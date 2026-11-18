---
name: qa
description: Execute already-built tests (or evaluate a set of pipeline/model outputs) and report what the evidence supports, with a verifiable audit trail and an explicit coverage statement. QA never declares pass/fail on its own authority — the human decides. Trigger on "run the tests", "QA this", "verify test results", "evaluate these outputs", "debug failing test", or "execute test suite". Assumes test construction (/create-test) is already done and trusted; this skill does not review test design.
disable-model-invocation: true
---

QA is execution and evaluation. It is not test design, and it is not a verdict.

Two rules govern everything below:

1. **Every claim must be checkable by the human.** If a statement in the report cannot be traced to a saved artifact and a locator the human can open, it must not be in the report as a finding.
2. **Every verdict is bound to its coverage.** QA reports on exactly what it inspected. It never extends a conclusion from inspected items to uninspected ones.

The final authority on whether something passes is always the human. QA reports evidence, gaps, and confidence. It does not issue "PASS".

---

## Step 1 — Define What Correct Means Before Running

State explicitly, before running anything:
- **What is being verified:** the production behavior or output set under evaluation, in plain terms
- **Success criteria:** specific, concrete conditions — the real outputs and state transitions expected, not "no errors"
- **Failure signatures:** what wrong output looks like, including subtle wrongness that would not crash anything

If these cannot be stated before running, stop and ask the user. Do not invent criteria after seeing the output.

---

## Step 2 — Declare Scope and Coverage Plan

Before running, state:
- **Population:** exactly how many items exist (N tests, N queries, N outputs). Count them; do not estimate.
- **Planned inspection:** which items will be inspected, at what depth, and how they were chosen.
- **Method**, one of:
  - **Full audit** — every item inspected. For large N, split into sequential batches and record progress per batch.
  - **Sample** — random or stratified. Record the seed or selection rule so the human can reproduce the sample. State what the sample can and cannot support.
  - **Delegated** — subagents each take a defined slice. Subagents require explicit user approval first. Record each agent's assignment (item IDs or range).

**Ask the user instead of choosing silently** when N is non-trivial and the user has not specified how thorough the QA should be. Lay out the options with their cost and what each one can support. If the user has already given clear instructions, follow them and restate the plan in one line.

**No coverage inflation.** "Inspected 5 of 100" means 5 verified and 95 not inspected. It never becomes a judgment on the 100. If the plan changes mid-run (time, volume), update the declared coverage and tell the user.

---

## Step 3 — Run and Save Raw Evidence

Run the test or collect the outputs. Save, before interpreting:
- the exact command(s) run, working directory, and relevant environment
- full raw output (stdout, stderr, logs) to files in an artifacts folder
- exit code(s)
- timestamps and, where relevant, the git commit or version under test

Default location: `qa-runs/<timestamp>/` containing `report.md` and `artifacts/`. Use the project's convention if one exists. The report must never be the only copy of the evidence.

---

## Step 4 — Read and Evaluate

Read the actual output. A clean exit code is not a result.

For each inspected item, ask:
- Does this output make logical sense given what was supposed to happen?
- Are the values, shapes, and content correct — not just present?
- Is anything in the logs inconsistent, suspicious, or silently wrong even though nothing crashed?
- Is the result coherent across cases, or does it only look right in the one inspected?

Assign each inspected item one state:
- **Verified correct** — checked against the Step 1 criteria, with a cited locator
- **Verified wrong** — expected vs. actual, with a cited locator
- **Inconclusive** — evidence is ambiguous or insufficient; say why
- **Not inspected** — in the population but not looked at

Tag every statement as one of:
- **Observed** — directly visible in a saved artifact, with a locator (file, line, item ID)
- **Inferred** — reasoning from observations; label it as such and keep it separate

Subagent results are claims, not evidence. Spot-check a portion of each agent's reported findings against the raw artifacts and record what was checked.

---

## Step 5 — When Something Is Wrong

If output is wrong, including subtle wrongness with no crash, report:
- Expected vs. actual, with locators into the saved artifacts
- The best current hypothesis for the cause, tagged as inferred
- What would confirm or refute the hypothesis

QA does not apply fixes. Report and stop; the user decides what happens next. If the user then approves a fix and it is applied, re-run from Step 3 and re-read the output, not only the exit code.

If the evidence suggests the test itself cannot prove what it claims (for example, it passes with the upstream producer disabled), do not diagnose the test design here. Report the evidence and hand it to `/create-test` as a finding.

---

## Step 6 — Escalate When Not Converging

Stop and report when:
- The same class of failure persists across 2–3 attempts with no new hypothesis
- The failure points to a wrong approach in the implementation, not a bug
- Progress would require a decision outside the approved scope

An escalation contains what was tried, the current failure, the hypothesis for why it persists, and the specific decision or information needed.

---

## Step 7 — Evidence Report

Write `report.md` containing:

1. **Coverage statement (first, before any findings):** population N; inspected n; method (full / sample with seed / delegated with assignments); per-state counts (verified correct / verified wrong / inconclusive / not inspected).
2. **Success criteria** from Step 1, verbatim.
3. **Findings**, each with state, observed/inferred tag, and locator.
4. **Anything that passed but warrants attention.**
5. **Confidence:** how certain the evidence makes each conclusion, what is *not* established, and what additional inspection would change that.
6. **How to verify me:** 3–5 specific claims from the report, each with the exact command or file location the human can use to check it independently. Include at least one claim of each kind that matters: one "verified correct" item, and, if any exist, one "verified wrong" item. Where practical, choose claims a skeptical reader would pick, not the easiest ones.

Do not summarize as "all tests passed" or "QA score X". State what was inspected, what was found, and what remains unverified. The human makes the pass/fail call.
