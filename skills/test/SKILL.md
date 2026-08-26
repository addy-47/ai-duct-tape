---
name: test
description: Execute, read, and evaluate tests that have already been constructed. Loops through runs, analyzes logs for silent wrongness (beyond exit code 0), and iterates on fixes until results are genuinely correct. Trigger on "run the tests", "verify test results", "debug failing test", or "execute test suite". Assumes /create-test discipline has already been applied.
---

## Step 0 — Confirm This Is a Valid Test to Run

Before running anything, check: does this test enter at the correct real-world entrypoint, or does it call a leaf function in isolation while the real system never invokes it that way?

If you're unsure, or if this test was not built using `/create-test`, stop and run that first. Do not execute and judge a test that couldn't have caught the bug it exists to catch — that produces false confidence, which is worse than no test.

---

## Step 1 — Define Success Before Running Anything

State explicitly:
- **Goal of this test**: what is being verified, in plain terms
- **Success criteria**: the specific, concrete conditions that mean this actually works — not "no errors," the real behavior expected
- **Failure signatures**: what wrong output would look like, including subtle wrongness that wouldn't crash anything

## Step 2 — Run and Read, Don't Run and Trust

**Do not accept a clean exit code as success.** Read the actual output.

For every result produced, ask:
- Does this output make logical sense given what was supposed to happen?
- Are the values, shapes, or content actually correct — not just present?
- Is there anything in the logs that is inconsistent, suspicious, or silently wrong even though nothing crashed?
- If this ran multiple times or on multiple cases — is the result coherent across all of them, or does it only look right in the case that was checked?

If output volume is large, sample deliberately across cases — not just the first result — and reason about what the full set implies.

**A test only passes when the actual result is verified correct against Step 1's success criteria — not when the process completed without throwing an error.**

## Step 3 — Issue Handling

If something is wrong — including subtle wrongness with no crash:

Report:
- What was expected vs. what actually happened
- Where in the logs/output this is visible
- Best hypothesis for the cause

Propose a fix. Apply only after approval. Re-run. Re-evaluate from Step 2 — not just re-check the exit code.

Repeat until the result is genuinely correct, not just non-crashing.

## Step 4 — When to Stop and Escalate (Not Loop Forever)

Stop and report instead of continuing to loop when:
- The same class of issue persists after 2-3 fix attempts with no new hypothesis
- The failure suggests the implementation's actual approach is wrong, not just buggy — this is an architecture question, not a test question
- Fixing the issue would require a decision outside the scope of what was approved
- The test itself may not be structurally valid (see Step 0) — if this surfaces mid-loop, stop and revisit via `/create-test` rather than continuing to iterate on a test that can't prove what it claims to

When stopping: state exactly what was tried, what the blocker is, and what you need from the user to proceed.

## Step 5 — Final Report

Only after genuine verification:
- What was tested and why that proves the goal
- What was actually observed in output/logs, not just "test passed"
- Any fixes applied during the loop
- Anything borderline that passed but is worth attention
- Explicit confidence statement: how sure are you this is actually correct, and why
