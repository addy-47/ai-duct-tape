---
name: modify-plan
description: Modify an existing implementation plan artifact mid-thread in response to new discoveries, requirement changes, blockers, or architecture adjustments. Use for "update the plan", "adjust the plan", "add a step", "pivot the plan". Not for creating new plans.
---

# Modify Plan

Read the existing implementation_plan artifact (baseline) and the new input (the modification trigger).

## Step 1 — Classify the change

| Scale | Examples | Response |
|---|---|---|
| **Minor** | Fix a wrong detail, reorder two steps, tweak a dependency | Make the edit, state what changed and why in 1–2 lines. No table, no full rubric. |
| **Moderate** | Add/remove a step, change scope of one phase | Short version of the rubric below — just the sections that actually apply. |
| **Major** | Pivot, architecture change, new requirement reshaping multiple phases | Full rubric below. |

## Step 2 — Edit the plan artifact in place

- Surgical edits only — change only what the trigger actually affects.
- Mark modified sections `[MODIFIED] <reason>`, added steps `[ADDED]`, removed steps `[REMOVED — reason]`.
- Never rewrite the whole plan to make a small change.

## Step 3 — Respond, sized to Step 1's classification

**Minor**: one or two lines — what changed, why. Done.

**Moderate / Major** — use what's relevant from:

1. **Trigger** — new requirement / architecture change / additional step / correction, in one line.
2. **What changes** — table: Location | Current | Proposed Change | Reason. (Only for Moderate/Major — skip entirely for Minor.)
3. **What doesn't change** — explicit, only if there's real risk of ambiguity about scope.
4. **Risks** — what could break, ordering constraints that shift, ⚠️ on high-uncertainty changes. Major only, unless something specific is actually at risk in a Moderate change.
5. **Assumptions** — "None" if none; otherwise flag explicitly.

Do not begin implementation after modifying — regardless of scale.
