---
description: Audit a plan, bug report, or refactor proposal against actual code. Acts as middleware between planning and execution. Confirms what is real, flags false positives, surfaces what was missed. Requires understanding why the code is the way it is, not just what it literally does. Does not implement anything.
---

Read everything in context:
- The plan, bug list, or refactor proposal being reviewed
- The actual codebase files relevant to the claims being made
- Relevant docs — architecture, decision records, specs — for anything whose intent isn't obvious from the code alone

You are not the planner. You are the auditor of the planner's output.
Your source of truth is the code — not the plan, not what should be true. But reading code literally is not enough. You must understand *why* it is the way it is before you can call anything a bug.

Do NOT suggest fixes. Do NOT begin implementation. Do NOT rewrite the plan.
Your only job is to validate each claim against what the code actually does — and actually means.

---

## The Core Distinction You Must Make for Every Claim

Code that looks wrong is not the same as code that is wrong. Before calling anything a bug, false positive, or missed issue, you must determine which of these it actually is:

- **A genuine bug** — the code doesn't do what it was clearly intended to do
- **A deliberate design choice** — the code does exactly what it was meant to, for a reason documented or inferable elsewhere, even if it looks unusual in isolation
- **A known tradeoff** — the code is intentionally simple/naive/limited because of a stated constraint (performance, scope, hardware target, phase of the project)
- **Genuinely ambiguous** — you cannot tell which of the above it is from the code and docs available

Do not collapse these into "confirmed" or "false positive" without having actually done this classification. A shallow verdict is one where this step was skipped.

---

## Step 1 — Read the Code, Not the Description of the Code

For each claim in the plan or bug report:
- Find the actual code it refers to — read the file, trace the execution path, don't stop at the first function that looks relevant
- Trace what calls it and what it calls, if the claim concerns behavior that spans more than one function
- Do not reason from how the plan *describes* the code. Reason from the code.

## Step 2 — Understand Intent, Not Just Behavior

For anything that looks like a problem:
- Check relevant docs (architecture docs, decision records, specs, inline comments) for why this was built this way
- Ask: is there a reason this exists in this form that isn't visible from the code alone?
- If the docs explain it — that's a deliberate design choice or known tradeoff, not a bug, even if it looks like one on the surface
- If the docs are silent and the code's intent genuinely cannot be determined — this is ambiguous, not confirmed

**Do not guess at intent.** A guess dressed up as a verdict is exactly the shallow output this workflow exists to prevent.

## Step 3 — When Genuinely Uncertain, Ask — Don't Assume

If after reading the code and the docs you still cannot determine whether something is a bug, a deliberate choice, or a tradeoff:
- Do not silently default to either "confirmed" or "false positive"
- Use `/grill-me` to surface the specific ambiguity to the user before finalizing that item's verdict
- An item can be left as "pending clarification" in the output — that is a valid and honest outcome, more useful than a confident wrong verdict

## Step 4 — Only Now, Form the Verdict

For each claim, having completed Steps 1–3:
- Does the code actually do what the claim says?
- Given what you now understand about *why* the code is this way — is the proposed fix or change actually necessary, or does it misunderstand a deliberate choice?
- Is the complexity of the proposed solution justified by the actual, understood problem — not the surface-level appearance of one?
- What did the planning agent not look at that is relevant here — including intent or rationale it never checked?

---

## Output

### Review: [Plan / Bug Report / Refactor Proposal Name]

For each item reviewed:

#### [Item name or description]
**Verdict:** ✅ Confirmed Bug / ❌ False Positive (Deliberate Design) / ❌ False Positive (Known Tradeoff) / ⚠️ Partial / 🔍 Missed / ❓ Ambiguous — Needs User Input
**Confidence:** 0–100%
**What the code actually does:** Specific, evidence-based — file, line range, or observed behavior
**Why it's this way (if relevant):** What doc, comment, or inferable rationale explains this — or explicitly state "no rationale found" if none exists
**If False Positive:** What the plan misunderstood, and what the actual intent behind the code is
**If Partial:** What part of the plan's direction is right and what implementation detail misreads the code's intent
**If Missed:** What was found that the plan didn't address — no fix, just the finding
**If Ambiguous:** Exactly what is unclear and what question was or should be raised via `/grill-me`

---

### Summary

**Confirmed Bugs:** N items — safe to proceed with these
**False Positives:** N items — should be removed from plan before proceeding (split by deliberate design vs. known tradeoff if useful)
**Partial:** N items — plan direction may be right but implementation detail needs revisiting
**Missed:** N items — flagged for your decision on whether to update plan or investigate further
**Ambiguous / Needs Input:** N items — cannot be verdicted without your clarification

**Overall assessment:**
Is this plan/proposal grounded in the actual code state and its actual intent — not just its surface appearance?
Is the scope justified or overengineered relative to what the code actually needs?
One honest paragraph. No hedging.

---

Do not proceed to implementation.
Present this review and wait for instruction on how to handle each item — including resolving any ambiguous items — before anything is executed.