---
name: mutate
description: Post-green validation protocol that empirically proves a test catches real regressions, rather than assuming it from reasoning alone. Seeds deliberate logical defects ("mutants") into production code, confirms the test suite goes red, then reverts and confirms green. Stack- and project-agnostic. Trigger on "validate this test", "how do I know this isn't a false green", "mutation test this", "prove this test catches bugs". Follows /create-test and /test — consumes the Phase 2b False-Green table as its input spec.
disable-model-invocation: true
---

A green test suite is a claim, not evidence. The only way to know a test actually fails when the logic it protects is broken is to break that logic on purpose and watch it fail. This skill turns the hypothetical false-green table from `/create-test` into real code edits and empirically confirms — or disproves — that the tests catch what they claim to catch.

**The governing question:**
> If I make this exact change to production code, does this specific test go red for the reason I predicted — and does it go back to fully green the moment I undo the change?

If either half fails, the test (or the mutant) is not what it appears to be.

This is a known, established technique called **mutation testing**. A deliberately introduced defect is a **mutant**. A test run against a mutant either **kills** it (fails, as it should) or the mutant **survives** (the suite stays green despite broken logic — a real coverage gap). **Mutation score = mutants killed / mutants total.**

---

## Precondition

This skill requires a test that has already passed `/create-test`'s Phase 4 circuit breaker and is currently green under `/test`. Its Phase 2b False-Green table is the mutant source list. If no such table exists, go generate one first — do not invent mutants from nothing; that produces exactly the kind of shallow, disconnected fault-injection this skill exists to avoid.

---

## Mutant Taxonomy — the shallow/logical distinction

**Discard these — they are not valid mutants for this protocol:**
- Panics, unwraps-on-none, `todo!()`/`NotImplementedError`, type errors, missing parameters, compile failures.
- These crash loudly. Virtually any test harness catches a crash. Killing one tells you nothing about whether the test observes the *right* thing — only that the process didn't explode. If a False-Green row would only produce this kind of mutant, rewrite the row until it describes silent wrong behavior instead.

**Use these — they change behavior without changing the shape of success (still returns Ok, still returns a value, no exception):**

| Category | What it does | Example |
|---|---|---|
| Silent drop | A call or side-effect is deleted, but the function still returns success | Delete the `send`/`publish`/`write` call but leave the surrounding `Ok(())` in place |
| Boundary/threshold flip | Comparison operator or constant is changed | `>=` → `>`, `0.85` → `0.5`, `contradiction ≥ threshold` → `≤ threshold` |
| Boolean/gate inversion | A guard condition is negated or hardcoded | `should_suppress()` body replaced with `return false;` unconditionally |
| Routing/destination swap | Output is sent to the wrong handler/channel/consumer, call shape unchanged | Route a result to handler B instead of the correct handler A |
| Priority/ordering inversion | A comparison or sequence used for conflict/priority resolution is reversed | `if incoming_priority <= matched_priority` → `>=` |
| Default/early-return fallback | A computed value is replaced with a hardcoded default | A classifier always returns its fallback category instead of the computed one |

The governing rule for picking mutants: **each row in the False-Green table should map to exactly one of these categories.** If you can't map a row to one, the row was too vague to begin with — go back and make it a concrete one-line code change before running this skill.

---

## Protocol

### 1. Extract
Pull mutants directly from the seam's False-Green table — at minimum one mutant per row. For each row, write down the exact one-line (or minimal) code edit that would realize it. Discard/rewrite any row that would only produce a shallow mutant (see taxonomy above).

### 2. Mutate
Make the single surgical edit in production code. One mutant at a time — never stack multiple mutations before testing, or you can't attribute a result to a specific defect.

### 3. Assert RED
Run **only** the target test file/module for that seam — never the full suite — using the project's normal test invocation, at production-equivalent build settings if the project distinguishes them (e.g. an optimized/release build, if debug builds behave meaningfully differently). Record which specific assertion failed and why.

- **Failed as predicted → mutant killed.** Move to step 4.
- **Passed → SURVIVOR.** This is a defect finding, not a shrug. Two possible resolutions, in order of likelihood:
  1. The test's assertion is too weak or checking the wrong thing — strengthen it, then re-run this same mutant to confirm it now kills it.
  2. The mutant is behaviorally equivalent to correct production code for this particular test's inputs (rare) — document why, and discard it from the mutant count rather than silently dropping it.

### 4. Revert & Assert GREEN
Undo the exact edit. Confirm the diff against version control is empty (e.g. `git diff --stat` clean, or your VCS's equivalent) before re-running the target test file and confirming it returns to full green. Do not proceed to the next mutant on top of an unreverted one — a poisoned baseline invalidates every subsequent result.

### 5. Score
Report per seam/boundary, not just an aggregate pass/fail:

```
Seam: [name]
Mutants attempted: N
Killed: K
Survivors: S (with row source, edit made, and remediation taken for each)
Mutation score: K/N
```

---

## Cost & Scope Discipline

Mutation loops multiply test runtime by mutant count. Scope every run tightly:

- Run only the single test file/module targeting the mutated seam — never the project's full suite per mutant.
- If the project's test runner parallelizes across shared/global state non-deterministically, force single-threaded execution for this loop to avoid attributing a race condition to the mutant.
- If any fixtures are expensive (model warm-up, live network calls, hardware access, container spin-up), prioritize mutating the cheap, pure-logic parts of that seam (gates, routing, priority/threshold comparisons) first. Save any broader automated sweep for last, and scope it explicitly — never run it unscoped across a codebase with expensive fixtures.

---

## Two Tiers

**Tier 1 — Primary, mandatory:** the manual, False-Green-table-derived mutants above. These are targeted at logic you already identified as load-bearing; they are cheap relative to their signal and directly answer "does this specific test catch this specific defect."

**Tier 2 — Secondary, optional:** an automated mutation-testing tool for the language in use (e.g. `cargo-mutants` for Rust, `mutmut`/`cosmic-ray` for Python, Stryker for JS/TS, PIT for Java/JVM). These generate mutants mechanically (arithmetic/comparison flips, deleted statements, boundary shifts) across a much larger surface than a human would think to check by hand, catching gaps Tier 1 missed. Use Tier 2 only:
- scoped to specific pure-logic files/modules, never the whole codebase in one run,
- against the seam's targeted test file, not the full suite,
- after Tier 1 is complete, as a supplementary net rather than a replacement.

Verify the exact invocation syntax for whichever tool applies to the stack in use — flags and defaults vary by tool version and are worth confirming against current docs rather than assumed.

---

## Handoff

Output the per-seam score table plus the survivor list with remediation. Any test strengthened or production defect found as a result feeds into the project's normal defect-tracking convention (RCA, ticket, etc.) — a survivor is a real finding, treated the same as any other bug this process surfaces.
