---
name: rca
description: Perform a structured root cause analysis on bugs, test regressions, flakes, or production incidents, and propose a fix scaled to what's actually found. Trigger on "why did this fail", "investigate this regression", "RCA on this error", "diagnose this flake", or "trace this bug".
---

# RCA

Read everything in context: what broke, error output/logs, the implementation plan and recent changes if available.

Do not propose a fix before Step 3 classifies the problem. Do not reason from memory alone if the codebase can be checked directly — grep, logs, actual execution path.

## Step 1 — Symptom

State precisely: observed vs. expected behavior, when it started (which change/deploy/event, if known), consistent or intermittent. If any of this can't be answered from context, stop and ask before continuing — don't guess at a timeline or trigger.

## Step 2 — Trace

Start at the symptom, not a hypothesis. Follow the actual execution path, not the intended one. At each step: could this be the source, or a consequence? Flag explicitly where your confidence drops.

## Step 3 — Classify

This determines everything that follows — get it right before writing the report.

| Class | What it means | What happens next |
|---|---|---|
| **Trivial** | Localized, obvious cause, no design implication (typo, off-by-one, bad default) | Root cause + fix in the same response. No stop. |
| **Contained** | Clear cause within one function/module, fix doesn't touch shared contracts | Root cause + fix in the same response, fix clearly marked, proceed unless user objects. |
| **Systemic** | Crosses module/service boundaries, violates an assumed invariant, or the fix could ripple | Root cause only. Stop. Wait for confirmation before proposing a fix. |
| **Environmental** | External to the code (config, infra, dependency version, data shape) | Root cause only, plus what to check/change outside the codebase. Stop if the fix requires access or changes you can't verify yourself. |
| **Confidence < 80%** | Regardless of the above | Always stop, regardless of class. State what information would close the gap. |

## Step 4 — Report

### Symptom
What broke, exactly.

### Root Cause
One clear statement. If multiple contributing causes, rank them.

### Classification
Which class from Step 3, and why — one line.

### Why It Wasn't Caught
Only if there's a real gap (missing test, missing validation, bad assumption). If nothing was actually missing — say so, don't invent one for the sake of the section.

### Confidence
0–100%. If below 80%, this already triggered a stop above — say what's needed to close the gap.

### Fix (Trivial / Contained only — inline here)
- Proposed fix — surgical, minimal
- Files and lines affected
- Why this addresses the root cause, not the symptom
- Regression risk
- How to verify — specific command or observable outcome

## Step 5 — Prevention (after fix confirmed working, if warranted)

What should be added (test, validation, guard) to prevent recurrence. Skip this section entirely if there's genuinely nothing worth adding — don't pad it.
