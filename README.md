# 🛠️ AI Duct Tape

> **The Source of Truth Library for Autonomous AI Agent Contexts**  
> A curated, modular library of battle-tested rules, personas, engineering skills, and code style guides designed for AI coding assistants.

Nothing in this repository is hardcoded to a single project or company product. Instead, **AI Duct Tape** is the foundational blueprint you copy into your project's agent directory (`.agents/`, `.opencode/`, `.claude/`, etc.) to turn standard LLMs into rigorous, disciplined, and proactive software engineers.

---

## 🏛️ The Four Pillars

```
ai-duct-tape/
├── rules/         📜 Global invariants & operational constraints (Rule Zero, Blocker Escalation)
├── roles/         🎭 7 specialized agent personas with invariant boundaries and ownership rules
├── skills/        ⚡ 24 structured workflows (/intent-alignment, /architect, /review, etc.)
└── style-guides/  📐 Stack-specific & general engineering standards (Design, TS, Rust, Go, Python)
```

| Pillar | Overview | Read More |
|---|---|---|
| **📜 Rules** | Universal constraints that govern every thread and model decision. Establishes Rule Zero (*Ask, Don't Assume*), the 2-Attempt blocker rule, and anti-hallucination policies. | [rules/README.md](rules/README.md) |
| **🎭 Roles** | Deep personas (Backend, Frontend, Architect, ML Research, QA, Test) equipped with domain mindsets, boundary-leak alerts, and explicit anti-goals. | [roles/README.md](roles/README.md) |
| **⚡ Skills** | 24 executable workflows triggered as slash commands (`/name`) that guide agents step-by-step through discovery, architecture, planning, refactoring, and review. | [skills/README.md](skills/README.md) |
| **📐 Style Guides** | Strict coding, API, and design standards ensuring consistent, high-performance, and accessible code across languages and frameworks. | [style-guides/README.md](style-guides/README.md) |

---

## 🔄 The Autonomous Engineering Lifecycle

AI Duct Tape aligns agent workflows along the entire software development lifecycle:

```mermaid
flowchart LR
    A[💡 Idea / Request] --> B[🎯 /intent-alignment]
    B --> C[📐 /architect & /create-spec]
    C --> D[📋 /create-plan]
    D --> E[🍰 /build-vertical]
    E --> F[🧪 /create-test & /test]
    F --> G[🔬 /review]
    G --> H[🤝 /handoff]
```

1. **Discovery & Alignment:** Pressure-test assumptions early (`/intent-alignment`, `/grill-me`).
2. **Architecture & Specification:** Formulate behavioral specs (`/create-spec`) and architecture docs (`/architect`).
3. **Phased Planning:** Draft detailed step-by-step execution plans (`/create-plan`, `/create-loop`).
4. **Execution & Implementation:** Implement thin end-to-end traces across all layers (`/build-vertical`, `/refactor-clean`, `/hotfix`).
5. **Rigorous Verification:** Construct structural tests and run iterative execution loops (`/create-test`, `/test`) along with adversarial senior code reviews (`/review`, `/feedback-review`).
6. **Session Handoff:** Package state cleanly for the next thread (`/handoff`).

---

## 📋 Copy Recipes (Quickstart)

Copy relevant files directly into your project's agent configuration folder. Do not submodule this repository.

### 🌐 Full-Stack Project (React/TypeScript + Backend)
```bash
rules/global-rules.md            → .agents/rules/global-rules.md
roles/*.md                       → .agents/rules/
skills/*/SKILL.md                → .agents/skills/<skill-name>/SKILL.md
style-guides/general.md          → .agents/rules/
style-guides/design.md           → .agents/rules/
style-guides/typescript-react.md → .agents/rules/
style-guides/<backend-lang>.md   → .agents/rules/
``` specialized agent personas with invariant boundaries and ownership rules

### ⚙️ Backend-Only Microservice (Rust / Go / Python)
```bash
rules/global-rules.md            → .agents/rules/global-rules.md
roles/backend-engineer.md        → .agents/rules/
roles/system-architect.md        → .agents/rules/
roles/qa-engineer.md             → .agents/rules/
roles/test-engineer.md           → .agents/rules/
style-guides/general.md          → .agents/rules/
style-guides/<language>.md       → .agents/rules/
```

### 🧠 Machine Learning & Data Evaluation
```bash
rules/global-rules.md            → .agents/rules/global-rules.md
roles/ml-research-engineer.md    → .agents/rules/
roles/qa-engineer.md             → .agents/rules/
skills/create-dataset/SKILL.md   → .agents/skills/create-dataset/SKILL.md
skills/create-eval/SKILL.md      → .agents/skills/create-eval/SKILL.md
style-guides/general.md          → .agents/rules/
style-guides/python.md           → .agents/rules/
```

---

## 🖥️ Agent Host Compatibility

| Host | Roles Placement | Skills Placement |
|---|---|---|
| **Antigravity** | `.agents/rules/` | `.agents/skills/` *(⚠️ filter host-specific content)* |
| **OpenCode (Project)** | `.opencode/` or `.agents/rules/` | `.opencode/skills/` |
| **OpenCode (Global)** | `~/.config/opencode/` | `~/.config/opencode/skills/` |
| **Claude Code** | `.claude/` | `.claude/skills/` |
| **Cursor / Windsurf** | `.cursor/rules/` or `.windsurfrules` | Workflows directory |

*(⚠️ Note: `skills/agy-subagent/` and `/schedule` in `roles/ml-research-engineer.md` are Antigravity-specific features.)*

---

## 🧭 Navigation for Agents

If you are an AI assistant browsing this repository, read **[AGENTS.md](AGENTS.md)** first for a direct lookup map linking task types to required reading files.

---

## 🤝 Attribution

Third-party skills, upstream origins, and vendored references (e.g. `grill-me`, `impeccable`) are documented in [THIRD_PARTY.md](THIRD_PARTY.md).
