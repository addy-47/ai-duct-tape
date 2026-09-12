---
name: test
description: Execute, read, and evaluate tests that have already been constructed. Loops through runs, reads output for silent wrongness beyond exit code 0, and escalates when the loop is no longer converging. Trigger on "run the tests", "verify test results", "debug failing test", or "execute test suite". Assumes /create-test discipline has already been applied.
disable-model-invocation: true
---

## Step 0 — Confirm This Is Worth Running

Before running anything, verify: does this test enter at a real production entry point, or does it call a leaf function that production never invokes directly?

If the test was not built using `/create-test`, or if the entry point looks like a convenience call rather than a genuine production seam, return to `/create-test` first. Running a structurally invalid test produces false confidence that is harder to undo than a missing test.

---

## Step 1 — Define What Passing Means Before Running

State explicitly:
- **What is being verified:** the production behavior this test covers, in plain terms
- **Success criteria:** the specific, concrete conditions that constitute correct behavior — not "no errors," the real outputs and state transitions expected
- **Failure signatures:** what wrong output looks like, including subtle wrongness that would not crash anything

If these cannot be stated before running, the test is not ready to run.

---

## Step 2 — Run and Read

Run the test. Then read the actual output — do not accept a clean exit code as the result.

For every result produced, ask:
- Does this output make logical sense given what was supposed to happen?
- Are the values, shapes, or content correct — not just present?
- Is anything in the logs inconsistent, suspicious, or silently wrong even though nothing crashed?
- If this ran across multiple cases: is the result coherent across all of them, or does it only look right in the case that was inspected?

If output volume is large, sample deliberately across cases — not just the first — and reason about what the full set implies.

**A test passes Step 2 only when the actual result is verified correct against the Step 1 success criteria.** A process completing without throwing an error is not that.

---

## Step 3 — When Something Is Wrong

If output is wrong — including subtle wrongness with no crash — report:
- What was expected vs. what actually happened
- Where in the logs or output this is visible
- The best current hypothesis for the cause

Then propose a fix, state explicitly what approval is needed to proceed, and wait. Do not apply changes without confirmation. After applying, return to Step 2 — re-read the output, do not only recheck the exit code.

Repeat until the result is genuinely correct against Step 1's criteria.

---

## Step 4 — Escalate When the Loop Is Not Converging

Stop iterating and report back when:
- The same class of issue persists after 2–3 fix attempts with no new hypothesis
- The failure points to a wrong approach in the implementation rather than a bug — this is an architecture question, not a test-loop question
- A fix would require a decision outside the approved scope
- The test itself may not be structurally valid — if the entry point looks wrong mid-loop, stop and return to `/create-test` rather than continuing to iterate on a test that cannot prove what it claims

A good escalation report contains: what was tried, what the current failure is, the hypothesis for why it persists, and the specific decision or information needed to unblock.

---

## Step 5 — Final Evidence Summary

Only after genuine verification against Step 1's criteria:
- What was tested and why this constitutes evidence for the stated goal
- What was actually observed in output and logs — not just "test passed"
- Any fixes applied during the loop
- Anything that passed but warrants attention
- An explicit confidence statement: how certain is this result correct, and what would change that assessment
