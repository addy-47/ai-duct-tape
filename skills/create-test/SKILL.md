---
name: create-test
description: Construct tests that are structurally capable of catching real production bugs before writing or running any test code. Identifies true entrypoints, upstream boundaries, streaming modes, and observable side effects to prevent vacuum tests. Trigger on "write a test for", "create test case", "design integration test", or "how should we test this component". Precedes /test.
---

A test that always passes is not a weak test — it is very often not testing anything real. Loading real weights, real files, or real data does not make something an integration test. The word "integration" refers to the interfaces, queues, threads, and state handoffs *between* subsystems — not to whether the inputs are realistic.

**The single question that governs this entire workflow:**
> If the real logic between this test's entrypoint and its target silently broke — dropped data, ignored a flag, mis-routed a message — would this test fail?

If the honest answer is no, the test is validating something in a vacuum. It does not matter how real the inputs are.

---

## Step 1 — Determine What Kind of Test This Actually Needs to Be

- **Unit test**: pure logic, no upstream producer, no cross-boundary handoff. Calling the function directly is correct and sufficient.
- **Integration test**: the thing being tested is never invoked this way in real operation — it receives its input from an upstream actor, queue, channel, event, or user action. If this is the case, continue below. Do not treat this as a unit test with bigger inputs.

State which one this is and why, before proceeding.

---

## Step 2 — Find the True Entrypoint (Integration Tests Only)

Do not call the target's own leaf method directly if, in real operation, nothing else ever calls it that way.

- Trace the actual real-world path backward from the target: what produces its input in production? A user action, an upstream actor, a queue push, an emitted event?
- If the target is fed by something upstream — **inject at that upstream boundary**, not at the target's convenient public method.
- If the target genuinely is the entrypoint in real operation (a true public API/IPC boundary with nothing upstream in-process) — injecting directly there is correct.

**The trap to avoid:** picking the entrypoint that's easiest to call in a test file, rather than the one that's actually used in production. Easiest-to-call and structurally-correct are frequently not the same function.

---

## Step 3 — Simulate Real Conditions, Not Convenient Ones

- If production feeds data in chunks, strides, or a stream — the test must do the same. A single monolithic input hides exactly the race conditions, gating logic, and state handoffs that make integration tests worth having.
- If the subsystem has more than one operating mode (e.g. continuous/passive vs. gated/manual-trigger, sync vs. async, single-user vs. concurrent) — each mode must be tested independently. A pass in one mode proves nothing about another. Do not assume coverage transfers across modes.
- Match realistic timing/pacing where it's structurally relevant (debounce windows, buffering thresholds) — not because it looks thorough, but because that's where real bugs actually live in event-driven systems.

---

## Step 4 — Assert on Real Observable Behavior

- For event/actor/channel-driven systems: subscribe to the actual event stream or channel output and assert on sequence, transitions, and final state — not on a synchronous return value that doesn't reflect how the system actually communicates.
- For simpler systems: assert on real side effects — persisted state, emitted output, actual downstream data — not a mock standing in for them.
- If the only way to assert something is to mock the exact boundary that matters, this is a signal you're testing the mock, not the system. Go back to Step 2.

---

## Step 5 — The Litmus Test (Mandatory, Before Finalizing)

Before this test is considered valid, answer explicitly:

> If the routing/handoff logic between the real producer and this consumer silently dropped data or ignored a state flag, would this specific test fail?

If yes — proceed to `/test` for execution and judgment.
If no — this test is a vacuum test. Return to Step 2 and find a higher-leverage entrypoint. Do not finalize a test that fails this question, no matter how much real infrastructure it touches.

---

## Step 6 — If the Code Itself Cannot Be Tested Correctly, Stop

This is not a workaround problem. If constructing a valid test at the correct entrypoint would require any of the following:

- Reaching into private/internal state that isn't exposed
- Writing bypass logic that doesn't correspond to any real production code path
- Mocking out a boundary that is a real queue/actor/channel in production, because the real one can't be observed or injected into

**Do not fabricate a workaround.** This means the code itself is not structured to be testable at the boundary that actually matters. Stop and report:

- Exactly which boundary cannot be observed or injected into
- What seam would need to exist (e.g. an injectable channel, an exposed hook, an observable event) for a valid test to be possible
- Flag this as an implementation task for whoever owns that code — this is an architecture/testability gap, not something a test can paper over

A test that quietly depends on logic that doesn't exist in the real path is worse than no test — it produces false confidence indefinitely.

---

## Handoff

Once a test passes the Step 5 litmus test, hand off to `/test` for the run/read/judge loop. This workflow's job ends at "this test is structurally capable of catching a real failure" — `/test` takes over from there.
