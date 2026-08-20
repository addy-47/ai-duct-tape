---
description: Rust code style guide and engineering standards. Agents doing write operations on Rust code should read this before modifying code.
---

This is the durable coding standard for Rust in this project's ecosystem. **Agents doing write operations must read this file before modifying Rust code.**

---

## 1. Module Organization & Types

- **Domain over type:** Group code by domain (`services/memory/nli.rs`), never by Rust construct (`models.rs`).
- **Single responsibility:** 1 responsibility per file. If a file cannot be described in 1 sentence, split it.
- **File size ceiling:** Flag and justify files exceeding ~600 lines. Do not let files grow silently past this.
- **`mod.rs` & `lib.rs`:** `mod.rs` is for module declarations + re-exports only. Zero business logic. `lib.rs` is for module declarations + application setup only. Zero business logic.
- **Visibility:** Use `pub(crate)` over `pub` unless crossing the crate boundary (IPC, plugin surface, or integration tests).
- **Trait & Generics Design:** Prefer static dispatch (`impl Trait` / generic type parameters) by default. Use dynamic dispatch (`Box<dyn Trait>`) only when heterogeneous collections or dynamic runtime polymorphism is strictly required.
- **Derives:** Prefer explicit `#[derive(Debug, Clone, PartialEq, Eq)]` on domain types and value objects where semantic equality and logging are needed.

## 2. Error Handling

- **No `unwrap()` in production code:** Banned except on lock guards where a poisoned lock is genuinely unrecoverable.
- **Propagation:** Use `?` with `.context("...")` (e.g. `anyhow`) in services and persistence layers.
- **Boundary errors:** Errors returned across process/IPC/API boundaries must be typed enums (e.g. `thiserror`).
- **No silent error swallowing:** Never `let _ = res`. Log discarded errors: `if let Err(e) = ... { tracing::warn!(...) }`.

## 3. Async & Concurrency

- **Non-blocking executor:** Never execute CPU-heavy work on async worker threads. Use `tokio::task::spawn_blocking` (or the runtime's equivalent).
- **Channels over locks:** Use async/crossbeam channels for inter-service communication. Avoid new `Arc<Mutex<T>>`.
- **Hot paths:** Hot paths (high-frequency, latency-critical) must be zero allocations and zero lock acquisitions. Use snapshotted values.

## 4. Linting & Verification

- `cargo check`: Mandatory zero errors.
- `cargo clippy --all-targets`: Mandatory zero warnings. Never suppress with `#[allow(...)]` without an explanatory comment.
- `cargo fmt`: Must be run before committing.

## 5. Testing & Benchmark Structure

| Category | File Location | Command | Access Scope | Primary Output |
|---|---|---|---|---|
| **Unit Test** | Bottom of target `.rs` file in `#[cfg(test)] mod tests` | `cargo test --lib` | Private + public functions | Pass / Fail |
| **Integration Test** | `tests/<feature>_test.rs` | `cargo test --test <name>` | Public crate API only | Structural Correctness |
| **Performance Benchmark** | `benches/<feature>_bench.rs` | `cargo test --bench <name>` | Custom `fn main()` (`harness=false`) | Latency & Throughput |
| **CLI Utility Tool** | `examples/<name>.rs` | `cargo run --example <name>` | Runnable dev tools | Standalone Utility CLI |

**Recommended header format for `tests/`, `benches/`, `examples/`:**
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

## 6. General Conventions

- **Constants:** No magic inline values. Constants go at the top of the file. Shared subsystem constants go in a dedicated constants module.
- **Secrets & Credentials:** Sensitive values live in environment files (never committed). Never hardcode credentials anywhere, including tests.
- **Dependencies:** Never add a new crate to `Cargo.toml` without explicit approval. Prefer the standard library and already-present deps.
