---
trigger: model_decision
description: Rust code style guide and engineering standards. Agents doing write operations on Rust code should read this before modifying code.
---

# Rust — Code Style Guide & Engineering Standards

This document establishes durable coding standards for Rust backend, systems, and service development across this ecosystem. **Agents doing write operations must read this file before modifying Rust code.**

---

## 1. Module Organization & File Boundaries

- **Domain over type:** Group code by domain (e.g. `services/memory/nli.rs`, `pipeline/audio/stt.rs`), never by Rust construct (`models.rs`, `types.rs`, `handlers.rs`).
- **Single responsibility:** 1 responsibility per file. If a file cannot be described in 1 sentence, split it.
- **File size ceiling:** Flag and justify files exceeding ~600 lines. Do not allow files to grow silently past this limit.
- **`mod.rs` & `lib.rs`:** `mod.rs` is for module declarations, re-exports, and **subsystem-level constants**. Zero business logic. `lib.rs` is for module declarations and root application/plugin wiring only. Zero business logic.
- **Visibility:** Use `pub(crate)` over `pub` unless crossing a crate/IPC boundary or exposing an API for integration tests.
- **Trait & Generics Design:** Prefer static dispatch (`impl Trait` / generic type parameters) by default. Use dynamic dispatch (`Box<dyn Trait>`) only when heterogeneous collections or dynamic runtime polymorphism is strictly required.
- **Derives:** Prefer explicit `#[derive(Debug, Clone, PartialEq, Eq)]` on domain types and value objects where semantic equality and logging are needed.

---

## 2. Constant Hierarchy & Placement (CRITICAL)

Never scatter or bury magic numbers or configuration values inside internal actor loops or worker functions. All constants follow a strict 4-level hierarchy:

1. **Global Constants (`core/constants.rs` or `constants.rs`):**
   - App-wide constants shared across multiple subsystems (e.g., `BUFFER_SIZE`, `DEFAULT_TIMEOUT_MS`, `DB_FILENAME`, system event names).
2. **Settings Defaults (`core/defaults.rs` or `defaults.rs`):**
   - Default values for user-configurable settings and catalog options (e.g., default provider, voice, temperature, retry limits).
3. **Subsystem / Domain Constants (top of that domain's `mod.rs`):**
   - Domain-specific thresholds, frame limits, buffer sizes, model paths, and worker queue depths live at the **top of that domain's `mod.rs`**.
   - Anyone inspecting a subsystem must immediately find its tuning parameters in `mod.rs` without searching through internal worker files.
4. **Single-File Internal Constants (top of specific `.rs` file):**
   - Constants purely local to a single struct or algorithm implementation (not shared across sibling modules) live at the very top of that specific file.

---

## 3. Function Standards & Code Cleanliness

- **Function line cap (soft):** No function should exceed 50 lines without documented justification.
- **Docstrings:** Exactly one `///` doc comment per function stating what it does, what it takes, and what it returns. Zero per-line comments inside function bodies. Runtime traces belong in `tracing::info!` / `tracing::warn!`.
- **No step-comment sequences:** If a function body requires numbered step comments (`// 1. do X`, `// 2. do Y`), each step must be extracted into a named private helper function.
- **No toggle functions:** A function named `connect()` must only connect. `if condition { connect } else { disconnect }` in one function body is banned. Use discrete named functions.
- **Struct bundling for parameter lists (>5 arguments):** Any function or constructor taking more than 5 arguments must group related parameters into a dedicated typed config or handles struct (e.g. `WorkerConfig`, `TelemetryHandles`).
- **Zero `#[allow(...)]` policy:** `#[allow(clippy::too_many_arguments)]`, `#[allow(dead_code)]`, `#[allow(unused_variables)]`, and other lint suppressions are strictly banned without explicit, documented justification.
- **Zero `_` prefixed masking:** Never prefix unused variables or fields with `_` to silence compiler warnings. If an item is not needed, delete it.
  - *RAII drop guard exception:* `_` is strictly reserved for genuine RAII drop guards (`_stream: Option<cpal::Stream>`, `_log_guard: Option<WorkerGuard>`, `_thread_handle`) where holding the handle in memory is required to keep hardware streams or workers alive.

---

## 4. Error Handling & Resilience

- **No `unwrap()` in production code:** Banned except on poisoned `RwLock`/`Mutex` guards where panic is unavoidable.
- **Propagation:** Use `?` with `.context("...")` (`anyhow` / `eyre`) in services and persistence layers.
- **Boundary errors:** Errors returned across IPC, RPC, or crate boundaries must be typed enums using `thiserror`.
- **No silent error swallowing:** `let _ = result` is banned. Every channel send or fallible call must either propagate with `?` or log warnings on failure:
  ```rust
  if let Err(e) = tx.send(item) {
      tracing::warn!("[Domain::Subsystem] Channel send failed: {}", e);
  }
  ```
- **No fallback chains:** Avoid `if path A fails, try path B, try path C`. One deterministic path per operation. If the path fails, report the error.

---

## 5. Concurrency, Threading & Hot Path

- **Actor-Engine Separation:** The actor owns the OS thread and state machine. The engine owns compute/inference logic. Never merge them into one monolithic struct or file.
- **Thread Placement:**
  - CPU-heavy or blocking tasks (e.g. model execution, heavy hashing, synchronous file I/O) run on dedicated OS threads or `tokio::task::spawn_blocking`. Never block async runtime workers.
  - IPC, network sockets, and event routing run on async runtime tasks (e.g. Tokio).
  - Dedicated worker threads must use elevated priority where real-time timing is critical.
- **Hot Path is Sacred:** High-frequency, latency-critical paths (e.g. audio pipelines, frame processing, message routing) must be zero allocations and zero lock acquisitions. Hot-path workers must read snapshotted/atomic values.
- **Channels over Shared Mutexes:** Cross-thread communication uses async/crossbeam channels or atomics. Avoid introducing new `Arc<Mutex<T>>`.
- **Canonical Lock Order:** When multiple locks must be acquired, establish and enforce a strict canonical acquisition hierarchy to prevent lock inversion deadlocks.
- **Event-Driven over Polling:** Subsystems must emit domain events when state changes rather than having callers poll atomic flags on a timer.

---

## 6. Production Rust Best Practices

- **Structured Logging:** All logs must specify domain tags: `tracing::info!("[Domain::Subsystem] Action completed status=ok")`. Never use raw `println!` or `eprintln!` in production code.
- **Dropped Counter Telemetry:** High-throughput channel `try_send` calls must increment an atomic dropped-counter handle and log warnings if backpressure occurs.
- **Newtype Pattern:** Prefer lightweight typed wrappers or domain aliases over raw primitives for identifiers (e.g. `TurnId(u32)`, `UserId(String)`).
- **Exhaustive Enums for State:** Model lifecycles using explicit state enums with transition functions rather than coordinating bags of loose booleans.

---

## 7. Testability Seams & Inversion of Control (MANDATORY)

Every backend actor, worker, pipeline domain, and router must be designed with explicit consideration of how it will be instantiated and tested in isolated unit and integration test harnesses:

1. **Generic Runtime / Platform Handles (`AppHandle<R: Runtime>` / Generic Contexts):**
   - Never bind actor functions, worker threads, or routers to concrete default GUI/platform runtimes. Parameterize with `<R: Runtime>` (or `R: Runtime + 'static` for spawned threads) so integration test suites can run headlessly with mock runtimes without requiring live OS windows or display servers.
2. **Decoupled Ingestion & Dispatch Seams (No Isolated Module Statics):**
   - Module-level statics (`static BUFFER: Mutex<...>`, `static IS_RUNNING: AtomicBool`) must never form isolated black boxes that upstream actors cannot feed or tests cannot observe.
   - Expose explicit ingress/egress seam functions (e.g. `ingest_data(&[T])`, `is_running() -> bool`, `handle_stop_with_sender(...)`) so tests can drive workflows without booting physical hardware drivers.
3. **Inversion of Control for Hardware & External Dependencies:**
   - High-level orchestrators that dispatch commands to downstream channels must support optional sender overrides or fallback gracefully when executing in headless test environments where physical hardware (e.g. audio/camera/sensor drivers) is absent.

---

## 8. Linting & Verification

Before declaring any Rust task complete, execute the standard verification pipeline:

- `cargo check`: Mandatory zero errors.
- `cargo clippy --all-targets`: Mandatory zero warnings. Never suppress with `#[allow(...)]` without an explanatory comment.
- `cargo fmt --check`: Mandatory clean formatting.

---

## 9. Testing & Benchmark Structure

For detailed testing taxonomy, benchmark execution standards, and evaluation guidelines, refer to **`style-guides/testing.md`**.

| Category | File Location | Command | Access Scope | Primary Output |
|---|---|---|---|---|
| **Unit Test** | Bottom of target `.rs` file in `#[cfg(test)] mod tests` | `cargo test --lib` | Private + public functions | Pass / Fail |
| **Integration Test** | `tests/<feature>_test.rs` | `cargo test --test <name>` | Public crate API only | Structural Correctness |
| **Performance Benchmark** | `benches/<feature>_bench.rs` | `cargo test --bench <name> --release` | Custom `fn main()` (`harness=false`) | Latency & Throughput |
| **CLI Utility Tool** | `examples/<name>.rs` | `cargo run --release --example <name>` | Runnable dev tools | Standalone Utility CLI |

**Mandatory header format for `tests/`, `benches/`, `examples/`:**
```rust
//! ============================================================================
//! <filename> — <one-line description>
//! ============================================================================
//! Category     : [Integration Test | Benchmark | Utility Tool]
//! Component    : <target module or subsystem>
//! Prerequisites: <required env vars, services, or data>
//! Execution    : <exact cargo command>
//! Metrics      : <recorded operational/quality metrics>
//! ============================================================================
```
