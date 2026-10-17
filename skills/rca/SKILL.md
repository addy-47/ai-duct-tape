---
name: rca
description: Debug and root-cause bugs, test failures, regressions, flakes, and production incidents with real evidence, then propose a fix scaled to what is actually found. Use for "why did this fail", "debug this", "this isn't working", "why is X happening", "investigate this regression", "RCA on this error", "diagnose this flake", "trace this bug", any error or stack trace, and any time a fix attempt has already failed once. Decides when to read and trace and when to add logs and reproduce instead.
---

# RCA

Read everything in context: what broke, error output/logs, the implementation plan and recent changes if available.

Do not propose a fix before Step 4 classifies the problem. Do not reason from memory alone if the codebase can be checked directly: grep, logs, actual execution path.

## Step 1 — Symptom

State precisely: observed vs. expected behavior, when it started (which change/deploy/event, if known), consistent or intermittent. If any of this can't be answered from context, stop and ask before continuing — don't guess at a timeline or trigger.

## Step 2 — Reproduce

Get the smallest failing reproduction before theorizing. Prefer one you can run yourself and repeat (a failing test, a script, a command) over manual clicking. Record the exact command and its output; this is the check every later step is measured against.
- Can't reproduce? That is the first finding. Say so, and aim Step 3's probes at capturing the failure where it actually happens.
- Only the user can reproduce it (their UI, device, environment, credentials)? Ask for one specific thing: the command to run, the value to paste, the screen to check.

## Step 3 — Trace, hypothesize, probe

Start at the symptom, not a hypothesis. Follow the actual execution path, not the intended one. At each step: could this be the source, or a consequence? Flag explicitly where your confidence drops.

Keep a short ledger of candidate causes. For each, write the observation that would confirm or rule it out. Take the next action that splits the most candidates.

**Choose how to find out: trace, probe, or ask.** Compare the cost of observing with the cost of being wrong.
- **Probe** (add targeted logs, run the reproduction, print the real value) when behavior depends on runtime state, timing, concurrency, or an external system; when reading hasn't revealed the cause; or when observing takes seconds and more reasoning takes minutes.
- **Think and trace** when the logic is deterministic, the path is short and readable, or probing would be destructive or expensive.
- **Ask the user** when only they can observe it, or when the answer is taste or product intent.

**Probe discipline**
- Put probes at boundaries (what goes into and comes out of each step), so one run shows where reality departs from your model. Aim for one run that answers the question.
- Tag every probe with a greppable marker, `[RCA-PROBE]`.
- Don't change what you are measuring: no logging inside timing-critical hot paths (for example, the audio callback or VAD loop in Vox). Use counters or sampled logs there. Never log secrets or personal data.
- Remove every probe when done, and confirm none remain by grepping the marker over the diff.

**Treat every fix as a hypothesis.** Before applying a fix you can't verify by reading, state the hypothesis and the observation that would confirm it. Confirm the fix by rerunning the same reproduction from Step 2.

**Stop circling.** After one failed fix, or two focused attempts, stop reasoning and instrument. Three tool calls with no new information, or the same area edited three times: stop and report **Known** (confirmed), **Unknown**, and **Next probe** or your question for the user.

## Step 4 — Classify

This determines everything that follows — get it right before writing the report.

| Class | What it means | What happens next |
|---|---|---|
| **Trivial** | Localized, obvious cause, no design implication (typo, off-by-one, bad default) | Root cause + fix in the same response. No stop. |
| **Contained** | Clear cause within one function/module, fix doesn't touch shared contracts | Root cause + fix in the same response, fix clearly marked, proceed unless user objects. |
| **Systemic** | Crosses module/service boundaries, violates an assumed invariant, or the fix could ripple | Root cause only. Stop. Wait for confirmation before proposing a fix. |
| **Environmental** | External to the code (config, infra, dependency version, data shape) | Root cause only, plus what to check/change outside the codebase. Stop if the fix requires access or changes you can't verify yourself. |
| **Confidence < 80%** | Regardless of the above | Probe first if a probe is cheap, safe, and reversible (Step 3). Stop only when the next probe needs the user's environment or access, would be destructive, or has already run and left a gap. When you stop, state what information would close it. |

## Step 5 — Report

### Symptom
What broke, exactly.

### Root Cause
One clear statement. If multiple contributing causes, rank them.

### Evidence
The reproduction command and the probe output or trace that supports the root cause. One or two lines.

### Classification
Which class from Step 4, and why — one line.

### Why It Wasn't Caught
Only if there's a real gap (missing test, missing validation, bad assumption). If nothing was actually missing — say so, don't invent one for the sake of the section.

### Confidence
0–100%. If below 80%, say what's needed to close the gap and whether a probe was tried first.

### Fix (Trivial / Contained only — inline here)
- Proposed fix — surgical, minimal
- Files and lines affected
- Why this addresses the root cause, not the symptom
- Regression risk
- How to verify — rerun the same reproduction, or give a specific command or observable outcome
- Probes removed: confirm no `[RCA-PROBE]` remains

## Step 6 — Prevention (after fix confirmed working, if warranted)

What should be added (test, validation, guard) to prevent recurrence. If the Step 2 reproduction can be kept as an automated test, say so. Skip this section entirely if there's genuinely nothing worth adding — don't pad it.