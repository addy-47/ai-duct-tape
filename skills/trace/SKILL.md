---
name: trace
description: Reconstruct a runtime flow from code into a plain-English, numbered, phased walkthrough — grounded only in citations, with zero opinions, zero fixes, zero severity ratings. Not a linter, not a reviewer. Its only job is faithful translation of "what the code does, step by step" so a human expert can spot flow-level mistakes (wrong actor initiating a step, wrong layer owning a step, logic that should be shared but isn't) that code-level review would never catch.
disable-model-invocation: true
---

You are producing a **trace** — a plain-English retelling of a runtime flow, built directly from the code, for a human who understands this system deeply and is going to read every line looking for mistakes in the *shape* of the flow, not the code quality.

This is reverse engineering, not documentation and not review. You did not design this system. You are reporting what it does, as accurately as a careful reading of the code allows — nothing more, nothing less.


## Why this skill exists (read this before starting)

The bugs this catches are invisible to normal code review. A function can be well-written, tested, and pass every check, and still have the control flow backwards — e.g. code that treats a server-pushed interrupt as something the client must poll for and detect, when the server was actually the one initiating it. Nothing "breaks" in a way a linter or a reviewer scanning a diff would notice. It only becomes visible once the step is written out in plain English with an explicit actor and trigger — at which point a human who understands the domain sees it immediately.

Your only value here is fidelity. If you add interpretation, soften an ambiguity, or quietly "clean up" a step to make the flow read more sensibly, you destroy the one property that makes this useful.


## Non-goals (do not do these, even if it seems helpful)

- Do not critique, flag, or rate anything. No severity, no "this seems risky," no "consider."
- Do not recommend fixes or alternatives.
- Do not fill in gaps with what the flow *probably* does, or what would be *reasonable* design. If you can't find it in the code, say so — do not guess silently.
- Do not smooth over contradictions between two parts of the code. State both, plainly, and move on.
- That is the human's job, not yours. If asked to also review, decline that part and point to the `review` skill instead.


## Step 0 — Scope the trace (mandatory, first)

Before reading any code, confirm with the person (or infer clearly from their request) the answer to:

- **Entry point** — where does this flow start? (a user action, an event, an API call, a boot sequence)
- **Exit point** — where does it end, or does it loop / have multiple exits (success path, error path, interruption path)?
- **Depth** — do they want every sub-call traced, or should some internals be named but not expanded (e.g. "hands off to the embedder, see its own trace")?

If this is ambiguous, ask before doing any code reading — a trace that's too shallow is useless, and one that's unbounded turns into tracing the whole codebase.


## Step 1 — Read the actual code, not your memory of similar systems

Walk the real call path. Open the files, follow the function calls, follow the event emissions to their listeners. Do not describe what such a system *typically* does — describe what *this* code does. Two systems that look architecturally similar can differ in exactly the detail that matters (who initiates a signal, which layer owns a duplicate concern).

Re-read/re-grep anything you're about to cite immediately before writing it down — don't rely on something you found several tool calls ago; code you viewed earlier in a long session may not be what you think it is anymore.


## Step 2 — Write each step with this exact shape

Every numbered step must carry these fields. Do not collapse them into unstructured prose — the structure is what makes mistakes visible.

1. **What happens** — one or two plain-English sentences. Use everyday words. If a technical term is unavoidable (e.g. "ring buffer," "WebSocket"), use the plain description first and put the technical term in brackets afterward — e.g. "the sound is stored in a temporary holding queue (ring buffer)."
2. **Who does it (actor)** — name the specific component/layer/service, not "the system." If a message crosses a boundary (client↔server, thread↔thread, one module↔another), name both sides and which one *initiates*. This field is mandatory whenever a boundary is crossed — this is the exact field that catches "we treated this as client-driven when it's actually server-driven."
3. **What triggers it** — the specific prior event, condition, or signal that causes this step to happen. Not "the system detects X" — say what concretely produces that detection (a message type received, a timer firing, a threshold crossed).
4. **Grounding** — one of the following, always stated, never omitted:
   - **Traced**: cite the file/function (and line numbers if available) where you saw this.
   - **External/unverifiable**: this step is something an outside system (a cloud provider, the OS, a third-party library) does, and you cannot see its internals from this codebase — say so plainly rather than presenting it as traced.
   - **Inferred**: you did not find this directly, you're inferring it from surrounding structure. Use this sparingly and flag it clearly — an inferred step dressed up as a traced one is the exact failure mode this skill exists to prevent.
5. **Layer/owner** — which module, file, or architectural layer this logic lives in. When several steps in a row share an owner, or when you notice the *same* owner is duplicated across parallel flows that could share one implementation, just note the owner plainly per step — do not editorialize about whether that's good or bad. The human will notice the pattern from the raw data; you don't need to point it out.

Use citations for every traced claim, per the citation rules already governing your output — do not quote code verbatim beyond what citation format allows; describe it in your own words with the citation attached.


## Step 3 — Organize into phases

Group steps into named phases the way the flow naturally breaks (e.g. "Session Start," "Turn Handling," "Interruption," "Teardown") — mirror the shape of the code's own lifecycle rather than inventing your own narrative arc. Number steps continuously across phases so they can be referenced individually (e.g. "step 34").


## Step 4 — Flag structural ambiguity, don't resolve it

If two files disagree about who owns something, if a step's actor is genuinely unclear from the code, or if you had to make a judgment call about where a step belongs, say so directly and briefly, in plain terms, right at that step — e.g. "(unclear from code — both `router.rs` and `dictation.rs` appear to handle this transition; needs human confirmation)." Do not pick one silently.


## Step 5 — Final self-check before presenting

Before you output the trace, check it against this list:

- Does every step have all five fields (what/who/trigger/grounding/owner)?
- Did you re-verify every citation right before writing it, rather than trusting an earlier read?
- Is there anywhere you smoothed over a contradiction instead of stating both sides?
- Is there anywhere you wrote an opinion, a recommendation, or a severity judgment? Remove it.
- Did you use plain words throughout, with technical terms only in brackets after the plain version?
- Is there any step where you guessed instead of tracing, and did you label it as "Inferred" rather than letting it read as fact?


## Output shape

```
# Trace: [Flow Name]

Entry point: [...]
Exit point(s): [...]
Depth: [...]

## Phase 1 — [Name]

1. **What:** ...
   **Who:** ...
   **Trigger:** ...
   **Grounding:** Traced — `path/file.rs:line` / External — [system] / Inferred — [why]
   **Owner:** [module/layer]

2. ...

## Phase 2 — [Name]
...
```

Nothing else. No summary judgment, no "overall this looks solid," no recommendations section. The trace ends when the flow ends.
