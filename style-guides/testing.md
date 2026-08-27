---
trigger: model_decision
description: Testing, evaluation, and benchmark standards across all stacks (unit, integration, evals, benchmarks). Agents authoring or executing tests must read this before acting.
---

# Testing, Evaluation & Benchmark Standards

This document establishes durable standards for designing, implementing, and running tests, evaluations (evals), and performance benchmarks across all projects and languages. **Agents authoring or running tests must read this file before acting.**

---

## 1. Testing Taxonomy & Scope

A complete testing strategy categorizes tests by their architectural boundary and purpose:

| Category | Typical Location | Execution Scope | Access Scope | Primary Output |
| :--- | :--- | :--- | :--- | :--- |
| **Unit Test** | Co-located or in test module next to target source (e.g. `#[cfg(test)] mod tests` in Rust, `test_*.py` in Python, `*.test.ts` in TS) | Fast, isolated | Private & public functions | Pass / Fail |
| **Integration Test** | Dedicated test directory (e.g. `tests/<feature>_test.rs`, `tests/integration/`) | Subsystem & cross-boundary flows | Public API / crate / package interface only | Structural & Lifecycle Correctness |
| **Evaluation (Eval)** | Dedicated evals directory (e.g. `evals/<capability>/`, `tests/evals/`) | Model, pipeline, or dataset evaluation | Public API + Model Inference + Datasets | Statistical Accuracy, WER, Judge Scores |
| **Performance Benchmark** | Dedicated bench directory (e.g. `benches/<feature>_bench.rs`, `benchmarks/`) | Hardware/runtime performance profiling | Isolated benchmark harness | Latency ($T_{\text{stage}}$, $T_{\text{E2E}}$) & Throughput |
| **CLI Utility Tool** | Examples or tools directory (e.g. `examples/<name>.rs`, `scripts/tools/`) | Interactive developer verification | Public API / Dev tools | Standalone Dev/Debug CLI |

---

## 2. Core Testing Principles

A test earns its place by covering behavior that could fail in production in a way that would matter.

- **Unit tests** earn their place by covering non-trivial algorithmic logic: state machine transitions, parsing edge cases, arithmetic boundaries, sanitization rules, and error paths. A unit test that only verifies struct default values or trivial conversions with no branching is measuring the compiler/interpreter, not the code.
- **Integration tests** earn their place by exercising real subsystem boundaries through public package APIs: handling dependency failures, buffer backpressure, concurrency races on shared state, or malformed upstream events. An integration test that calls a leaf function directly, bypassing the queues, actors, or channels that connect it to the rest of the system, is a unit test in disguise.
- **Evaluations (Evals)** earn their place by measuring statistical distributions, precision/recall, and semantic correctness on probabilistic or LLM-judged outputs against curated ground truth fixtures.
- **Performance benchmarks** earn their place by measuring real pipeline latency on realistic workloads: audio clips through full pipelines, queries against populated indexes, or requests under concurrent load. Measuring isolated struct serialization in a tight loop produces numbers that do not map to user-observable latency.

---

## 3. Benchmark & Evaluation Execution Standards

To prevent misleading results and invalid performance claims:

1. **Sequential Execution (Never Concurrent Inference/Benchmarking):**
   - Run benchmark probes one configuration/model at a time. Concurrent inference or CPU/GPU-bound loops cause thread contention and hardware throttling that invalidate latency comparisons.
2. **Optimized Build Configuration (Release Mode):**
   - Always run evaluations and benchmarks using optimized release profiles (e.g. `cargo test --bench <name> --release`, `cargo run --release`, or equivalent compiled optimization flags). Unoptimized debug builds omit vectorization, inlining, and compiler graph optimizations, producing latency numbers up to 7× slower than production.
3. **Per-Stage Latency Decomposition:**
   - Record and report per-stage latency (e.g. $T_{\text{ingest}}$, $T_{\text{process}}$, $T_{\text{dispatch}}$, $T_{\text{E2E}}$), not only end-to-end duration. A passing E2E time with a regressed internal stage is a hidden performance defect.
4. **Parametrizable CLI Probes:**
   - Author benchmarks and evals to accept CLI arguments (flags for fixture files, concurrency levels, iterations, or thresholds) so engineers and CI can probe realistic workloads without editing code or recompiling.
5. **Ground Truth Verification Standard for Probabilistic / Model Outputs:**
   - Assert normalized string similarity (e.g. $\ge 0.90$ Levenshtein) or Word Error Rate ($\text{WER} \le 0.10$) against clean, labelled ground truth fixtures. Asserting the presence of 1–2 keywords is insufficient — it fails to distinguish a correct output from a partially hallucinated one and misses regressions that alter meaning while preserving keywords.

---

## 4. Testability Seams & Inversion of Control

Production code must be designed to be testable without requiring brittle hacks, global mutable state, or live external hardware:

1. **Parameterize Runtime & App Handles:**
   - Never tightly bind domain workers or actors to concrete platform runtimes (e.g. concrete GUI windows, live webview event loops, or active display servers). Parameterize runtimes or use generic handles so test harnesses can drive the system headlessly.
2. **Decoupled Ingestion & Dispatch Seams (No Isolated Module Statics):**
   - Module-level statics must never form isolated black boxes that tests cannot observe or drive. Expose explicit ingress/egress seam functions (e.g. `ingest_buffer(&[T])`, `is_active() -> bool`, `dispatch_with_sender(...)`) so tests can drive workflows directly without booting physical hardware devices (microphones, audio output, physical sensors).
3. **Inversion of Control for External Dependencies:**
   - High-level orchestrators that dispatch commands to external services, hardware drivers, or network channels must support optional channel overrides or mock/headless fallbacks for deterministic, isolated test execution.

---

## 5. Mandatory Header Format

Every dedicated test file in `tests/`, `evals/`, `benches/`, or `examples/` should include a standard header:

```text
============================================================================
<filename> — <one-line description>
============================================================================
Category     : [Integration Test | Evaluation | Benchmark | Utility Tool]
Component    : <target module or subsystem>
Prerequisites: <required models, env vars, fixtures, or running services>
Execution    : <exact test / run command>
Metrics      : <recorded operational / quality metrics>
============================================================================
```

---

## 6. The Testing Lifecycle: `/create-test` → `/test` → `/mutate`

Every test authored in the codebase must pass through this rigorous 3-step discipline:

1. **Construction (`/create-test`):**
   - Trace the production entry seam and observable exit.
   - Run the Direction Check: ensure the test invokes the upstream trigger, not the output consumer.
   - Fill out the Phase 2b False-Green Table predicting 2–5 concrete defects that must cause the test to fail.
2. **Execution (`/test`):**
   - Define exact correctness criteria before running.
   - Run the test and inspect raw output/logs for silent wrongness beyond exit code 0.
   - Escalate when failing loops do not converge within 2–3 attempts.
3. **Mutation Validation (`/mutate`):**
   - Seed deliberate, minimal logical defects ("mutants") from the False-Green table into production code.
   - Empirically assert that the test goes **RED** (mutant killed) and returns to **GREEN** upon reverting the edit.
   - Calculate and report the seam's Mutation Score ($K/N$).
