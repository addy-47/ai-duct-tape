---
description: Go code style guide and engineering standards. Agents doing write operations on Go code should read this before modifying code.
---

This is the durable coding standard for Go in this project's ecosystem. **Agents doing write operations must read this file before modifying Go code.**

---

## 1. Tooling & Formatting

- **`gofmt` is non-negotiable.** All code must be `gofmt`-clean; this is enforced in CI. Prefer `goimports`/`gofumpt` for imports and stricter formatting if the project uses them.
- **`go vet`** must pass with zero findings.
- **`golangci-lint`** (or the project's configured linter) must pass with zero warnings. Never suppress with `//nolint` without an explanatory comment.
- **No new dependencies without explicit approval.** Prefer the standard library and already-present deps.

## 2. Idiomatic Go

- **Errors are values.** Handle errors explicitly with `if err != nil`. Never ignore an error with `_ =`, never `panic` for expected conditions.
- **Wrap errors with context:** `fmt.Errorf("...: %w", err)`. Preserve the original error in the chain.
- **Export only what's needed:** lower-case unexported identifiers by default; export when the package boundary requires it.
- **`context.Context` is first parameter** for anything that can block or be cancelled. Never store a context in a struct.

## 3. Module Organization

- **`cmd/` for binaries, `internal/` for non-importable code, `pkg/` only for code intentionally shared across the module.**
- **Domain over type:** group packages by domain, not by construct (`models`/`utils`). One responsibility per package. If a package cannot be described in one sentence, split it.
- **Small interfaces are a feature.** Define interfaces at the consumer, accept them, return concrete types.
- **No global mutable state** — inject dependencies (structs, constructors) explicitly.

## 4. Concurrency

- **Prefer goroutines + channels, and finish before the function returns.** A leaked goroutine is a bug.
- **Synchronize with `sync` primitives only when a channel is the wrong tool.** If you introduce a `Mutex`, you should be able to justify why a channel/ownership model didn't fit.
- **Guard against goroutine leaks in tests:** always cancel contexts, close channels, and `WaitGroup.Wait()`.
- **Never block the runtime** with unbounded work on a single goroutine without a plan for backpressure.

## 5. API & Services

- **Validate all input at the boundary** before it touches business logic. Never trust incoming data.
- **Never return raw DB errors or stack traces to the client** — sanitize all error responses.
- **Every list endpoint has pagination.** Never return unbounded queries.
- **Multi-table operations use a transaction.** Not optional.
- **One shared DB connection pool per service.** Never a connection per request.
- **Graceful shutdown:** drain in-flight requests before exiting.
- **Health check endpoint from day one** (`/health` or `/healthz`).

## 6. Linting & Verification

- `gofmt -l .`: must return nothing.
- `go vet ./...`: zero findings.
- `golangci-lint run`: zero warnings.
- `go build ./...`: zero errors.

## 7. Testing

- **Table-driven tests** are the default pattern for unit tests.
- **Naming:** `<func>_<behavior>` (`TestCalculate_ZeroInput`, `TestAuth_ExpiredToken`).
- **Benchmarks** live in `_test.go` files with `func BenchmarkXxx(b *testing.B)`.
- **Benchmarks run sequentially, never in parallel** — concurrent execution invalidates latency numbers.

## 8. General Conventions

- **Constants:** No magic inline values. Named constants at the top of the file; shared constants in the owning package.
- **Secrets & Credentials:** Sensitive values live in environment files (never committed). Never hardcode credentials anywhere, including tests. Provide `.env.example` with placeholders.
- **Logging:** structured logging with levels (info / warn / error). No scattered `fmt.Println` in production paths.
