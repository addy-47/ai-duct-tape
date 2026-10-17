---
name: create-test
description: Pre-implementation reasoning protocol for constructing tests that catch real production bugs. Stack- and project-agnostic. Traces the production path, verifies testability, and stress-tests for false-green paths before a line of test code is written. Trigger on "write a test for", "create test case", "design integration test", or "how should we test this component". Precedes /test and /mutate.
---

A good test is a thin wire connecting a real production entry point to a real observable output, such that if anything on that wire broke — dropped data, wrong routing, skipped state transition — the test fails. A good test contains no logic of its own. It wires up production components, invokes the entry point, and reads the output.

**The governing question:**
> If the production logic between this test's entry point and its observable output silently broke — dropped data, ignored a flag, mis-routed a message — would this test fail?

Hold this question throughout every phase. It is the single standard a test must meet.

This skill is stack-agnostic. The phases and the governing question are invariant across languages and frameworks; adapt file layout, test-runner invocation, and language idioms (Rust, Python, TypeScript, Go, JVM, whatever the project uses) to the codebase at hand.

---

## Phase 0 — Classify

State which one this is and the one-sentence reason before proceeding:

- **Unit test**: pure algorithmic logic, no upstream producer, no cross-boundary handoff. Calling the function directly is the correct entry point.
- **Integration test**: the behavior is driven in production by an upstream actor, queue, channel, event, request, or user action — not by calling a function directly. Continue below.

If difficulty reaching the correct integration boundary tempts a switch to a simpler, more callable component: that difficulty is the finding. A unit test of a downstream leaf is not a substitute for an integration test. If the real boundary cannot be reached, move to Phase 2 and report the testability gap.

---

## Phase 1 — Production Path Trace

Read the actual source code — not documentation — and write this before touching a compiler or test runner:

```
SUT (System Under Test):
  [the connected production behavior being verified — not a function name]

Production Entry Seam:
  [the real boundary where this behavior enters production and where
   production-owned state begins — not the most convenient callable API]

Direction Check:
  Is the Production Entry Seam the upstream TRIGGER of this behavior,
  or is it the output INTERFACE of the system under test?

  If the function you are calling to initiate the test is the same function
  that production calls to deliver a result, you are not testing the
  pipeline — you are testing the output consumer. Example: testing an
  order pipeline by directly invoking the function that sends the
  confirmation email tests the sink, not the trigger (placing the order).
  Find the real upstream trigger before proceeding.

Production Path:
  [upstream trigger: user action / API call / emitted event / message]
  → [producer / handler that owns ingestion]
  → [production-owned buffer / queue / channel / state]
  → [routing / dispatch / state machine]
  → [consumer / processor / downstream handler]
  → [observable output: event emitted, state persisted, side effect]

Observable Exit:
  [what the test asserts on — real downstream output from production,
   not a synchronous return value that bypasses the communication boundary]

Production functions the test will call:
  setup:   [fn/method — production constructor or factory]
  entry:   [fn/method — the real entry seam invocation]
  observe: [channel / event / callback / persisted state the test reads]

Functions I will write in the test file:
  [list each helper — a helper is valid only if it can be replaced by a
   direct call to an existing production function with no changes to
   production code; if it cannot, it belongs in production, not the test]
```

This trace is the contract for Phases 2–4. Nothing written in the test may contradict it.

---

## Phase 2 — Pre-Implementation Checkpoint

Two checks, both required before any test code is written.

### 2a — Testability

Answer all four. If any answer is No, the test cannot be validly constructed.

1. Can the Production Entry Seam be invoked from the test using its real production signature?
2. Can production-owned state be assembled by calling real production constructors — not by manually reconstructing equivalent state in the test?
3. Can the required channels, queues, callbacks, or persisted state be observed from the test?
4. Is there a real observable exit that does not require mocking the boundary being tested?

A blocked entry seam — regardless of whether the block is a visibility constraint, a type mismatch, a runtime requirement, or an initialization dependency — is a testability gap. The correct response is to report what is blocked, why it is blocked, and what seam (injectable channel, decoupled interface, exposed hook) would make a valid test constructible. Relocating the test to a more accessible but different entry point is not a resolution; it is a different test for a different thing.

### 2b — False-Green

For the proposed test, identify 2–5 ways it could pass while production is broken. The first row is mandatory for every integration test:

| If this production defect existed | Would this test fail? |
|---|---|
| **[upstream producer completely silent / removed]** | **must fail** |
| [message / event dropped] | must fail |
| [wrong destination / mode routed] | must fail |
| [production buffer/state never populated] | must fail |
| [add SUT-specific defect] | must fail |

> **Mandatory upstream bypass check:** If the upstream producer (the real trigger identified in Phase 1) were completely silent or removed, would this test still pass? A "yes" means the test is wired to the output of the system, not the input. It is a false-green by construction. Redesign before proceeding.

A "maybe" or "no" in the second column means the test design does not cover what it claims to cover. Redesign before proceeding.

**Keep this table.** It is not throwaway reasoning — it is the direct input to `/mutate`, which turns each row into a real code mutation and empirically confirms the test actually fails the way this table predicts. Write rows specific and mechanical enough that each one could be turned into a concrete one-line code change.

**The production reimplementation signal:** If the Phase 1 "functions I will write" list contains a function that runs a loop, manages state, dispatches commands, or implements the logic of a production component — that function is not test scaffolding. It is a reimplementation of production inside the test. The test will pass regardless of whether the production component it replaces works or not, which makes it a false-green by construction. That function belongs in production; the fact that it cannot be called from there is the testability finding.

---

## Phase 3 — Implementation

Write the test using the Phase 1 trace as the spec and the Phase 2 checkpoint as the constraint.

- Call real production constructors and factory functions to assemble components.
- Invoke the Production Entry Seam with real input — not a leaf method chosen for accessibility.
- Feed data the way production feeds it: in the operating mode being tested, at the correct granularity.
- Each distinct operating mode (e.g. passive vs. explicit-trigger, sync vs. async, single vs. batch) is its own test file — behavior in one mode does not validate another.
- Assert on the real observable exit: emitted events, persisted state, channel output, side effects.
- When the test fails because production dropped data, misrouted a message, or skipped a state transition: that failure is a finding. Report it (RCA, bug ticket, whatever the project's convention is). The test is working correctly.

**Negative assertions are mandatory for suppression, gate, and guard paths.** For any SUT that includes a suppression gate (rate limiting, ownership/permission gates, silence/no-op guards, feature-flag gates), at least one test must assert that the gated output is *absent* when the gate is active. "The happy path works" is not sufficient. "The gate correctly blocks the unhappy path" is the more important invariant. Wait deterministically and then check for absence — never use a short timeout as a proxy for "nothing happened," since that conflates "didn't happen yet" with "will never happen."

**Shared test infrastructure rules:**
- Helpers shared across test files belong in a common/shared test-support location (adapt to your project's convention — e.g. `tests/common/` in Rust, `conftest.py` fixtures in Python, `test-utils/` in TS). Each seam/boundary test file imports from it rather than duplicating setup.
- A valid helper function is one that can be replaced by a direct production call with no production code changes. If it cannot be, it belongs in production, not the test.
- Tests that require external credentials or live third-party services must be explicitly marked skip/ignore per your framework's convention (e.g. Rust `#[ignore]`, pytest `@pytest.mark.skip`, Jest `.skip`), with a comment on how to run it manually. Credentials load from the project's designated secrets mechanism — never hardcoded in the test file. Never run these in the automated/default loop.

**One test file per seam.** One primary test function covers the full happy path and is the pass/fail gate for that seam/boundary. Additional test functions in the same file cover edge-case guards (empty input, cancellation, absent-event assertions). Each test function must be independently runnable and must not depend on execution order or leftover state from another test (watch for shared global/static state — reset it explicitly in setup/teardown).

---

## Phase 4 — Circuit Breaker

Before submitting, verify the test against the Phase 1 contract:

A test passes this circuit breaker when:
- Its entry point matches the Production Entry Seam in the trace (not a downstream leaf).
- The Direction Check passes: the entry point is the upstream trigger, not the output receiver.
- Every node in the Production Path is exercised, not bypassed.
- Every function written in the test file passes the Phase 1 validity check (replaceable by a direct production call).
- The test level matches what was requested.
- All suppression/gate paths have at least one negative-assertion test.

Then answer the governing question one final time, concretely:
> If [specific handoff X] in the production path silently broke, this test would fail because [specific assertion Y] would not receive [expected value Z].

If you cannot complete that sentence with a real assertion, the test does not yet meet the governing question. Return to Phase 1.

---

## Handoff

When Phase 4 passes, hand to `/test` for execution. If the test fails because a production boundary is broken or disconnected, that is a successful outcome — the test found a real bug. Report it.

Once the test is green, its Phase 2b false-green table becomes the direct input to `/mutate`, which empirically proves (rather than assumes) that the table's predictions hold.
