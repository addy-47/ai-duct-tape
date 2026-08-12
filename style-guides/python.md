---
description: Python code style guide and engineering standards. Agents doing write operations on Python code should read this before modifying code.
---

This is the durable coding standard for Python in this project's ecosystem. **Agents doing write operations must read this file before modifying Python code.**

---

## 1. Tooling & Environments

- **Virtual environments:** Always use `venv` (or `uv`) for any Python service or script. Never install packages globally unless explicitly asked.
- **Project config:** Prefer `pyproject.toml` for packaging, tool, and dependency configuration. Minimize scattered `setup.cfg`/`requirements.txt` fragments.
- **Package manager:** Use the project's declared tool (pip/uv/poetry). Never add a dependency without explicit approval.

## 2. Style & Formatting

- **PEP 8 baseline.** Format with `ruff format` (or `black`), lint with `ruff check`.
- **Naming:** `snake_case` for functions/variables, `UPPER_CASE` for module-level constants, `PascalCase` for classes.
- **Line length:** ≤ 88 characters (ruff default). Long signatures are fine; long logic chains are not.

## 3. Typing

- **Type hints are mandatory** on all public function signatures and data structures. Bare `Any` is discouraged — use `type`/`TypeVar`/`Protocol`/generics where they add information.
- Run `mypy --strict` (or the project's configured type checker) as part of verification.

## 4. Module Organization

- **Domain over type:** group by domain (`services/payments/gateway.py`), not by construct (`utils.py`, `models.py`).
- **Single responsibility:** 1 responsibility per module/file. If a module cannot be described in 1 sentence, split it.
- **Small modules are a feature:** prefer several small, well-named modules over one large one.
- **No circular imports:** import modules, not functions, where it helps; keep shared types in dedicated modules.

## 5. Error Handling

- **Raise, don't return sentinels** for genuine failure. Use `Optional`/`Result`-style returns only where the absence is a valid value, not an error.
- **Never bare `except:`.** Catch the specific exception you can actually handle.
- **Log, don't print.** Use the logging module with structured levels (info / warn / error). No scattered `print()` statements.
- **Boundary sanitization:** never leak raw DB errors, stack traces, or internal messages to clients — sanitize all error responses.
- **Validate all input at the boundary** before it touches business logic. Never trust incoming data.

## 6. Async & Concurrency

- **Never block the event loop:** CPU-bound or blocking-I/O work runs in a thread/process pool (`asyncio.to_thread`, `run_in_executor`) or a worker queue.
- **Use async where the stack is async:** prefer `async def` + `await` over threads for I/O-bound concurrency in a single service.
- **Share state via messages/queues, not shared mutable globals.**

## 7. Linting & Verification

- `ruff check`: mandatory zero warnings.
- `mypy --strict` (or project type checker): mandatory zero errors.
- `ruff format --check`: must pass before committing.

## 8. Testing

| Category | File Location | Command | Scope |
|---|---|---|---|
| **Unit Test** | `tests/` mirrors the package, or alongside the module (`test_<name>.py`) | `pytest tests/` | Pure functions + isolated units |
| **Integration Test** | `tests/integration/` or `tests/<feature>/` | `pytest tests/integration/` | Public API / service boundaries |
| **Fixture Data** | `tests/fixtures/` | — | Deterministic inputs, never live data |

- Tests must be deterministic — no sleeps, no network, no ordering dependence without an explicit reason.
- **Exit code 0 is never sufficient evidence.** Read the output; verify correct values, not just "no exception."

## 9. General Conventions

- **Constants:** No magic inline values. Module-level constants at the top of the file; shared constants in a dedicated `constants.py`/config.
- **Secrets & Credentials:** Sensitive values live in environment files (never committed). Never hardcode credentials anywhere, including tests. Provide `.env.example` with placeholders.
- **Dependencies:** Never add a new package without explicit approval.
