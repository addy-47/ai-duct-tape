---
name: create-plan
description: Create a phased, actionable implementation plan from an approved architecture or spec before writing code. Plans Phase 1 in detail while keeping later phases high-level. Use at the start of implementation or when the user says "create a plan", "how should we implement this", "plan the rollout", or "break this into implementation phases".
---


Read everything in context:
- ROLE defines your constraints and behavior
- CONTEXT tells you current project state and source of truth
- PLAN is the architecture/"what" plan — your primary input
- TASK defines the scope

Your job is to produce an implementation plan where only the immediate phase is detailed.
The rest is intentionally high-level — it will be detailed after each phase completes via /modify-plan.

## Rules
- Plan only. No code. No implementation.
- No assumptions — if something is not in the provided docs or plan, flag it explicitly.
- If anything is ambiguous and would block Phase 1, stop and ask before continuing.
- Do not invent architecture, infer file structures, or guess at behavior.
- Do NOT plan later phases in detail — their reality depends on what Phase 1 actually produces.


## Output

### Implementation Plan: [Task/Phase Name]

**Objective**
One sentence. What this entire task delivers when complete.

**Source of Truth Used**
Every doc/file referenced from context to build this plan.

**Assumptions Made**
None — or flag each one explicitly.


### Phase 1 — [Name]
*(Fully detailed — this is the only phase planned in depth)*

**Goal:** What this phase delivers in isolation.

**Steps:**
1. [File or system] — [exactly what changes and why]
2. ...

**Validation before proceeding:**
- [ ] [specific check: build / test / manual / lint]
- [ ] ...

**On completion:**
Run `/review` to evaluate what was actually done in this phase.
Then run `/modify-plan` to update Phase 2 into full detail based on reality.
Do not proceed to Phase 2 without this step.


### Phase 2 — [Name]
*(Intent only — will be detailed after Phase 1 completes)*

- Goal: [one sentence]
- Expected inputs from Phase 1: [what this phase depends on]
- Success looks like: [observable outcome]


### Phase N — [Name]
*(Intent only)*

- Goal: [one sentence]
- Success looks like: [observable outcome]


*(Repeat intent block for each subsequent phase)*


**Ordering Constraints, Risks, and Scope**
Surface these naturally — not as rigid sections but as what the human needs to know before approving this plan. Flag anything with ⚠️ that has high uncertainty or could cascade if wrong.


Do not begin implementation.
Present this plan and wait for explicit approval.
