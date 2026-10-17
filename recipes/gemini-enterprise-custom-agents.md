# Recipe: Developing and Deploying Gemini Enterprise Custom Agents

A battle-tested recipe for developing, containerizing, authenticating, and deploying custom AI agents to Google Cloud Run for integration into **Gemini Enterprise (A2A protocol)**.

---

## 1. System Architecture

```
┌─────────────────────────────────┐
│     Gemini Enterprise Web UI    │
└────────────────┬────────────────┘
                 │ 1. End-User enters prompt
                 │ 2. OAuth sign-in flow (User grants Workspace scopes)
                 │ 3. A2A POST with Authorization: Bearer <user_oauth_token>
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│ Cloud Run Service (ADK + FastAPI / Uvicorn)                     │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ EnterpriseUserTokenMiddleware                             │  │
│  │ - Intercepts Bearer <token>                               │  │
│  │ - Stashes in builtins._adk_global_user_oauth_token        │  │
│  │ - Wraps response.body_iterator (keeps token active during │  │
│  │   SSE streaming tool calls)                               │  │
│  └─────────────────────────────┬─────────────────────────────┘  │
│                                │                                │
│                                ▼                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ ADK Agent Execution & Tools (agent_source.app.*)          │  │
│  │                                                           │  │
│  │ ├─ Vertex AI (Gemini 2.5) ───► ADC (Cloud Run Service SA) │  │
│  │ └─ Workspace APIs (Drive/Gmail/Cal) ─► User OAuth Token  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Gemini Enterprise OAuth Authentication

When a custom agent accesses user Workspace data (Gmail, Drive, Calendar, Docs, Sheets):
- **Service Accounts do NOT have Drive storage quotas** and **cannot access user Gmail accounts** without domain-wide delegation.
- Therefore, the agent **must use the signed-in end-user's OAuth token**.

### Step 2.1: GCP OAuth Client ID Setup
1. In **Google Cloud Console** -> **APIs & Services** -> **Credentials**:
   - Create an **OAuth 2.0 Client ID** (Type: *Web application*).
   - Add Authorized redirect URIs:
     ```
     https://vertexaisearch.cloud.google.com/oauth-redirect
     https://vertexaisearch.cloud.google.com/static/oauth/oauth.html
     ```
2. Note your `Client ID` and `Client Secret`.

### Step 2.2: OAuth Scopes & Authorization URI
When adding the agent authorization in the Gemini Enterprise Console:
- **Authorization URL**:
  ```
  https://accounts.google.com/o/oauth2/v2/auth?access_type=offline&prompt=consent
  ```
  *(Important: `access_type=offline` is strictly required by Google authorization servers).*
- **Token URL**:
  ```
  https://oauth2.googleapis.com/token
  ```
- **Scopes** (space-separated):
  ```
  openid email profile https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/gmail.modify https://www.googleapis.com/auth/drive https://www.googleapis.com/auth/spreadsheets
  ```
- **PKCE Verification**: Enabled.

---

## 3. ADK Runtime Pitfalls & Golden Implementation

### Pitfall A: ContextVar Module-Splitting
Under Uvicorn / ADK, the entrypoint runs as `__main__`, but ADK tools import via `agent_source.app.services...`. Standard module-level `ContextVar` instances get duplicated.
**Fix**: Anchor the token to Python `builtins`.

```python
# app/services/request_context.py
import builtins
from contextvars import ContextVar

if not hasattr(builtins, "_adk_global_user_oauth_token"):
    builtins._adk_global_user_oauth_token = ContextVar(
        "adk_global_user_oauth_token", default=None
    )

_user_oauth_token: ContextVar[str | None] = builtins._adk_global_user_oauth_token

def set_request_user_token(token: str | None) -> None:
    _user_oauth_token.set(token)

def get_request_user_token() -> str | None:
    try:
        return _user_oauth_token.get()
    except LookupError:
        return None
```

### Pitfall B: FastAPI Streaming Response Lifecycle
FastAPI `BaseHTTPMiddleware` finishes when headers are returned, before response chunks are yielded. Tool calls run while chunks are streamed.
**Fix**: Wrap `response.body_iterator`.

```python
# agent.py
class EnterpriseUserTokenMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        from app.services.request_context import set_request_user_token

        auth = request.headers.get("authorization") or request.headers.get("x-forwarded-authorization", "")
        token = auth[7:].strip() if auth.lower().startswith("bearer ") else ""
        token_val = token or None

        set_request_user_token(token_val)
        try:
            response = await call_next(request)
        except Exception:
            set_request_user_token(None)
            raise

        if hasattr(response, "body_iterator"):
            orig_body_iterator = response.body_iterator

            async def wrapped_body_iterator():
                set_request_user_token(token_val)
                try:
                    async for chunk in orig_body_iterator:
                        yield chunk
                finally:
                    set_request_user_token(None)

            response.body_iterator = wrapped_body_iterator()
            return response

        set_request_user_token(None)
        return response
```

### Pitfall C: Split Credentials (Vertex AI vs Workspace)
- **Vertex AI (`google-genai`)**: Requires the Cloud Run service account ADC (`allow_user_token=False`). The end-user token lacks the `cloud-platform` scope.
- **Workspace APIs (Drive/Gmail)**: Requires the end-user OAuth token (`allow_user_token=True`).

```python
# app/services/auth_service.py
def resolve_credentials(allow_user_token: bool = True):
    if allow_user_token:
        request_token = get_request_user_token()
        if request_token:
            return Credentials(token=request_token, scopes=SCOPES)
    creds, _ = google.auth.default(scopes=SCOPES)
    if getattr(creds, "requires_scopes", False):
        creds = creds.with_scopes(SCOPES)
    return creds
```

---

## 4. Models & Vertex AI Configuration

In production on Cloud Run, use Vertex AI. Gemini 3.x developer preview models (`gemini-3.1-flash-lite`) do not exist on Vertex AI. Use the **Gemini 2.5 family**:
- Chat / Reasoning: `gemini-2.5-flash` or `gemini-2.5-pro`
- Image Generation: `gemini-2.5-flash-image` (or Imagen 3)

Deploy with environment variables:
```bash
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=<PROJECT_ID>
GOOGLE_CLOUD_LOCATION=us-central1
MODEL=gemini-2.5-flash
IMAGE_MODEL=gemini-2.5-flash-image
```

---

## 5. Agent Card Specification (`agent.json`)

The Agent Card must match A2A protocol specification (v0.3):

```json
{
  "protocolVersion": "0.3",
  "name": "my_agent",
  "description": "Assistant description for Gemini Enterprise.",
  "url": "https://my-agent-<PROJECT_NUMBER>.<REGION>.run.app/a2a/agent_source",
  "version": "1.0.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false,
    "stateTransitionHistory": false
  },
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["text/plain", "image/png", "image/jpeg"],
  "skills": [
    {
      "id": "my_skill",
      "name": "Skill Name",
      "description": "Describes what the skill does.",
      "tags": ["education", "workspace"],
      "examples": ["Help me do X"],
      "outputModes": ["text/plain"]
    }
  ]
}
```

---

## 6. Dockerfile & Container Best Practices

```dockerfile
FROM python:3.11-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential curl \
    && rm -rf /var/lib/apt/lists/*

COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /srv/agent_source

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project

COPY . .

EXPOSE 8080
CMD ["uv", "run", "--no-sync", "python", "agent.py"]
```

---

## 7. Cloud Build & Cloud Run CI/CD

Deploy script / Cloud Build step:
```yaml
steps:
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'bash'
    args:
      - -c
      - |
        gcloud run deploy my-agent \
          --image="us-central1-docker.pkg.dev/$PROJECT_ID/adk-agents/my-agent:latest" \
          --region="us-central1" \
          --platform="managed" \
          --port=8080 \
          --allow-unauthenticated \
          --update-env-vars="GOOGLE_CLOUD_PROJECT=$PROJECT_ID,GOOGLE_CLOUD_LOCATION=us-central1,GOOGLE_GENAI_USE_VERTEXAI=TRUE,MODEL=gemini-2.5-flash,IMAGE_MODEL=gemini-2.5-flash-image"
```

---

## 8. Troubleshooting Checklist

| Symptom | Cause | Solution |
|---|---|---|
| `403 storageQuotaExceeded` on Drive | Request fell back to Service Account | Fix ContextVar singleton with `builtins` + wrap `response.body_iterator`. |
| `400 failedPrecondition` on Gmail | Request fell back to Service Account | Same as above; verify Bearer token is forwarded. |
| `The authorization URI must contain 'access_type=offline'` | Google OAuth provider constraint | Append `?access_type=offline&prompt=consent` to Auth URI in Gemini Enterprise. |
| Model 404 / NotFound on Cloud Run | Using `gemini-3.1-*` on Vertex AI | Set `MODEL=gemini-2.5-flash` in Cloud Run env vars. |
| Multiple duplicate images rendered | Intermediate assets appended to media list | Only append the final composite asset in tool execution. |
