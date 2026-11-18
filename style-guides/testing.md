---
trigger: model_decision
description: Testing, evaluation, and benchmark standards across all stacks (unit, integration, evals, benchmarks). Agents authoring or executing tests must read this before acting.
---

# Testing, Evaluation & Benchmark Standards

This is the binding standard for every test, evaluation (eval), and benchmark in the ecosystem. **Agents authoring or running tests must read this file before acting.** A test that does not meet it is incomplete, even if it passes.

The governing question:

> If the production logic between this test's entry point and its observable output silently broke — dropped data, ignored a flag, mis-routed a message, or failed to adapt to real-world skew — would this test fail?

The answer must be an immediate yes with a named assertion. Otherwise the test is decoration.

**Division of labour:** this guide owns the durable "what". `/create-test` owns the procedural "how" of test design. `/qa` owns execution and the audit trail. `/mutate` owns post-green regression proof. Skills link here; they do not restate this guide.

---

## 1. Levels

Levels are defined by what they prove, not by where the file lives.

| Level | Proves | Entry point |
| :--- | :--- | :--- |
| **Unit** | One algorithm, state machine, or boundary calculation is correct in isolation. | The function itself |
| **Component** | One production component behaves correctly through its public API, with no upstream actor. | The component's public interface |
| **Integration (seam)** | A real upstream trigger drives a real handoff across a real boundary, and the observable exit is correct. | The production trigger |
| **End-to-end** | The user-visible loop works: input → pipeline → output → idle. | The harness entry |
| **Evaluation** | Model output quality meets a threshold. Never inside an integration test. | The project eval harness |
| **Benchmark** | Latency or throughput meets a budget. | The benchmark harness |

Rules:

1. A test lives at the lowest level that can still reach its real trigger. If calling a leaf is correct, it is a unit test — do not fake a pipeline around it.
2. If production drives the behavior through an actor, channel, event, request, or user action, a unit test of the downstream leaf is not a substitute. That is a false green.
3. Never assert model quality inside an integration test. Integration asserts wiring: right payload, right channel, right order, non-empty output, proper exit condition.
4. A test that only asserts a row count or a callback count, without proving the state transition, is a plumbing contract test, not a user-facing behavior test.
5. A unit test that verifies only default values or trivial conversions with no branching measures the compiler, not the code.

---

## 2. File Layout: One File Per Seam

A test file compiles and links as its own unit. Compilation is a dominant cost, so file count is a budget, not a preference.

- One test file per production seam, containing 10–20 subtests.
- A subtest covers one behavior, one scenario, one invariant.
- A new behavior gets a new subtest, not a new file.
- Do not split a seam across files because the file is long. Split only when the seam genuinely splits into two production boundaries.
- Do not merge two seams because their tests look similar. The seam is the unit of ownership.
- A seam file that cannot reach 10 subtests is either under-tested or not a seam. Investigate before accepting it.

---

## 3. Intent Block — Mandatory in Seam Files

Every test function in a seam file carries the intent block. It is the durable output of `/create-test` and what a reviewer reads instead of guessing.

```text
PURPOSE:  The production defect this test exists to catch, in one sentence.
          Name a failure mode, not a feature.
SCENARIO: Preconditions and the exact user or system action.
ASSERTS:  The observable exit and the exact invariant. Every claim here
          must map to an assertion in the body.
CLASS:    happy-path | edge-case | error-path | suppression | regression | contract
REQUIRES: models, env vars, network, or time budget. Write "none" if none.
MUTANT:   The one-line production change that must turn this test red.
```

Binding rules:

1. `PURPOSE` states what breaks in production if the test is absent. "Verifies X works" is banned.
2. `ASSERTS` is falsifiable without reading the body. A reviewer can name the assertion line it maps to.
3. `CLASS` is mandatory. `suppression`, `edge-case`, and `regression` also require `MUTANT`.
4. If the block names a metric, threshold, or count, the body contains that assertion.
5. `MUTANT` is a concrete one-line edit, not a description. It is the seed consumed by `/mutate`.

A file header may document the seam boundary, not the business logic:

```text
<filename> — <seam name and the production boundary it covers>
Category     : Integration (Seam) | Component | E2E | Evaluation | Benchmark
Component    : <production modules in the path>
Prerequisites: <models / env / services, or "none">
Execution    : <exact command>
Metrics      : <recorded operational / quality metrics>
```

---

## 4. Naming

- Use the project's identifier case. Do not add a `test_` prefix.
- Pattern: `<subject>_<scenario>_<outcome>`.
- One behavior per function.
- Banned: `_matrix`, `_validation`, `_works`, `_basic`, `_sanity`, `_check`.
- A subtest is named for what it asserts, not what it touches.
- If the name needs "and", it is two tests.

---

## 5. Assertion Rules

### 5.1 Absence and suppression

Absence is proven only after quiescence is proven. A timer is not proof. A single non-blocking try-receive is not proof.

Preferred forms, in order:

1. **Capture-channel interception** — attach a sink before the boundary and assert nothing arrived.
2. **Producer closed** — drop the sender, then drain the receiver until disconnected.
3. **Barrier or acknowledgement** — a sentinel that proves the producer finished, then assert the queue is empty.
4. **Storage count** — assert a zero count after the transaction commits.

Banned: a sleep followed by a try-receive; any helper that does this; "no event within N ms" as the sole negative assertion.

A suppression test that cannot name its quiescence mechanism is not written yet.

### 5.2 Error paths

Every fallible seam asserts both directions: the failure is surfaced, and the happy path still works. An error test that asserts only an error exists does not prove it is the right error. Assert the variant or the message.

### 5.3 No swallowed failures

- A discarded join handle is banned. Join and assert that the worker did not panic.
- A silent drain loop is banned. Drain explicitly and assert on what was drained, or give each scenario its own actors.
- A background worker that dies must fail the test, not be absent from the assertions.
- Every non-fatal catch block logs a diagnostic. Empty catch blocks are banned.

### 5.4 Time

- Every async test wraps its body in a timeout. A deadlock must fail, not hang.
- Synchronous wait loops use a deadline or timeout primitive.
- No wall-clock latency assertion belongs in the default tier. `< 15 ms` on a shared machine measures the scheduler, not the product. Latency belongs in benchmarks or an explicitly marked benchmark subtest.
- Sleep is not a way to reach a state. Wait on the state transition or the event.

### 5.5 Ground truth

Accuracy assertions use a real, normalized, agreed metric against labelled fixtures. Single-keyword presence is banned. For probabilistic or model output, assert normalized string similarity (for example ≥ 0.90) or word error rate (for example ≤ 0.10) against clean ground truth.

---

## 6. Shared Helpers

- Anything used by two tests lives in a shared test-support module or fixture directory.
- A helper is valid only if it can be replaced by a direct production call with no production change. Otherwise it is production logic hiding in the test.
- A helper that reimplements a production predicate is a false green: test and production drift, and the test still passes. Extract the predicate to production and call it.
- Duplicated setup between seam files is deleted, not tolerated.
- Every helper carries a one-line comment stating what production contract it stands in for.

---

## 7. Execution

### 7.1 Local execution is the default

Production-grade tests are run locally on demand. CI is a compile guard, not an execution proof.

1. A green CI check does not mean the suite passes.
2. Every agent that changes test-covered code runs the affected seam locally and states the command and result.
3. A seam not run locally in the current session is reported as `unverified`, never as passing.
4. CI is compile-only by deliberate choice. It stays that way until the project says otherwise.

### 7.2 Run settings

- Run one seam at a time, sequentially, in the optimized release profile where the stack distinguishes profiles.
- Version the exact command in the project's own tooling or docs. Do not guess flags from memory.
- If the runner parallelizes across shared or global state, force single-threaded execution so a race is not attributed to a mutation or a code change.
- Report the command, the environment, and the result. "Passing" alone is not a result.

### 7.3 Testability seams and inversion of control

Production code must be testable without brittle hacks, global mutable state, or live external hardware.

1. **Parameterize runtimes and platform handles.** Never bind workers or actors to a concrete GUI runtime or display server. Generic handles let harnesses drive the system headlessly.
2. **Decoupled ingestion and dispatch.** Module-level statics must not form black boxes tests cannot observe. Expose explicit ingress and egress seam functions.
3. **Inversion of control for external dependencies.** Orchestrators that call services, drivers, or networks accept optional channel overrides or headless fallbacks.

---

## 8. Benchmark and Evaluation Execution Standards

To prevent misleading results and invalid performance claims:

1. **Sequential execution.** Run one configuration or model at a time. Concurrent inference or CPU/GPU-bound loops cause contention and throttling that invalidate latency comparisons.
2. **Optimized build.** Run evals and benchmarks with the optimized release profile. Debug builds omit vectorization and inlining and can report latencies several times slower than production.
3. **Per-stage latency decomposition.** Record per-stage latency, not only end-to-end duration. A passing end-to-end time with a regressed internal stage is a hidden defect.
4. **Parametrizable CLI probes.** Benchmarks and evals accept flags for fixtures, concurrency, iterations, and thresholds so workloads can be probed without editing code.
5. **Ground truth verification.** See section 5.5. Keyword presence is not a quality metric.

---

## 9. External Dependencies

- Cloud keys are not required by the default suite. Live keys buy contract verification; they cost determinism, money, rate-limit flakes, and secret handling.
- Offline tests intercept at the network boundary and assert method, URL, headers, body shape, auth presence, timeout, retry count, and error mapping.
- Recorded cassettes cover parser drift.
- Live tests are marked skip or ignore per the project's framework, run manually, and carry an owner, a reason, and an expiry date.
- An ignored test with no owner, no reason, and no expiry is deleted, not ignored.
- Never log secrets, even in test debug paths.

---

## 10. Mutation

- `MUTANT` in the intent block is the seed. It is a concrete one-line edit to production code.
- Mutation runs are batched per campaign over the changed files, not per test.
- A test whose predicted mutant survives is not done. Either the test is weak or the mutant was wrong. Resolve before merge.
- Production code is reverted after every mutation run. A mutant is never committed.
- Only a failure on the predicted assertion counts as a kill. A build error, setup crash, timeout, or flake invalidates the run.
- See `/mutate` for the full protocol and the mutant taxonomy.

---

## 11. Fold, Prune, Rename

A revamp is not additive. For every new or strengthened test, decide the fate of the old one.

| Situation | Action |
| :--- | :--- |
| Old test asserts a subset of the new one | Fold into the new subtest; delete the old function |
| Old test asserts the same thing through a weaker seam | Prune |
| Old test name overstates what it does | Rename to the real behavior, or fix the body |
| Old test passes if production is deleted | Delete, and record the finding |
| Old test covers a behavior nothing else covers | Keep; add the intent block |

No test survives on the grounds that it is already passing. Passing is not coverage.

---

## 12. Findings: Do Not Fix Production

When a test exposes a production defect:

1. Do not patch production to make the test pass.
2. Do not weaken the assertion.
3. Record the defect in the project's findings artifact with the test name, symptom, evidence, and suspected `file:line`.
4. Continue with the current seam.

A red test that found a real bug is a successful outcome. Hiding it is the failure.

---

## 13. Blocker Protocol

1. **Isolate immediately.** When a test fails, run the single subtest with the project's runner. Never loop the whole suite while debugging one failure.
2. **Trace with checkpoints.** Place explicit markers before and after locks, channel operations, and async boundaries.
3. **Two-attempt rule.** After two targeted hypotheses without a confirmed new one, stop and report the exact test, failing line, logs, both hypotheses and their results, and the blocker.
4. **No manufactured pass.** Never mock, stub, widen visibility, or delete an assertion to get green.

---

## 14. Checklist Before a Test Is Called Done

- [ ] It is at the lowest level that can still reach the real trigger.
- [ ] The entry point is the upstream trigger, not the output sink.
- [ ] The intent block is present and every `ASSERTS` claim maps to a line.
- [ ] Every suppression path has an absence test with a named quiescence mechanism.
- [ ] No discarded joins, silent drains, bare sleeps, or wall-clock latency asserts.
- [ ] Helpers live in shared test support and could be replaced by production calls.
- [ ] The name states the behavior, not the file's shape.
- [ ] `MUTANT` is present for suppression, edge-case, and regression classes.
- [ ] The seam was run locally; the command, environment, and result are reported.
