---
name: modify-plan
description: Modify an existing implementation plan artifact mid-thread in response to new discoveries, requirement changes, blockers, or architecture adjustments. Use whenever the user asks to "update the plan", "adjust the plan", "add a step to the plan", or "pivot the plan". Do not use to create new plans from scratch.
---


Read everything in context:

- The existing implementation_plan artifact is the baseline
- The new input (requirement, architecture change, or additional step) is the modification trigger

Do NOT proceed to modify anything yet.

## Step 1 — Impact Summary ( similar to this )

Before touching the plan, report back:

### Modification Trigger

What is the new input and what type of change is it?

- [ ] New requirement
- [ ] Architecture change
- [ ] Additional step
- [ ] Correction / fix

### What Changes in the Plan

For each affected phase or step:

| Location | Current | Proposed Change | Reason |
| -------- | ------- | --------------- | ------ |

### What Does NOT Change

Explicitly list phases/steps that remain untouched.

### Risks & Regressions

- What could break as a result of this modification
- Any ordering constraints that shift
- Flag high-uncertainty changes with ⚠️

### Assumptions Made

If none: state "None". If any: flag explicitly.


Stop here. Present this report and wait for explicit approval before modifying the plan.
Use the "Modification Trigger" , "What Changes in the Plan", "What Does NOT Change"," Risks & Regressions\* and "Assumptions Made" naturally as a extension of the report to user , they are not strict ouput formats they are more for yout internal thought process structuring .


## Step 2 — Plan Update (only after approval)

Update the implementation_plan artifact in place:

- Do not rewrite the entire plan
- Surgical edits only — change only what the impact report identified
- Mark modified sections clearly with `[MODIFIED]` and a one-line reason
- Mark any newly added steps with `[ADDED]`
- Mark anything removed with `[REMOVED — reason]`

Present the updated plan in full after edits are complete.
Do not begin implementation.
