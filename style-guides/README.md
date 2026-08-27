# Style Guides

Per-language and cross-cutting engineering standards. **Agents doing write operations must read the relevant guide before modifying code.**

## Guides

| File | Scope |
|---|---|
| `general.md` | Language-agnostic best practices — applies to every stack: modularity, API standardization, CLI discipline, security, Docker, `.gitignore`. Read this **always**. |
| `testing.md` | Cross-stack testing, evaluation, and benchmark standards: taxonomy, execution rules (sequential, release builds), per-stage latency, probabilistic ground truth thresholds, and testability seams. |
| `design.md` | Stack-agnostic UI/UX & design standards: design tokens, typography, copy architecture (`<xxx>Copy.ts`), layman language rules, gesture contracts, interactive states, accessibility (WCAG AA), and CSS conventions. |
| `typescript-react.md` | TypeScript / React frontend code architecture and state management |
| `rust.md` | Rust backend/systems/services code |
| `python.md` | Python services, ML pipelines, and scripts |
| `go.md` | Go services and APIs |


## How to Use

1. Always read `general.md` — it applies regardless of language.
2. Read `testing.md` whenever designing, implementing, or running tests, evals, or benchmarks.
3. Read the language guide for the stack you're writing.
4. Guides are meant to be **copied into a consuming project** (e.g. `.agents/rules/` or a project `docs/` dir) — see the repo root `README.md` for copy recipes.

## Structure Template

New language guides follow this shape:

```markdown
---
description: <Language> code style guide and engineering standards. Agents doing write operations on <Language> code should read this before modifying code.
---

1. **Tooling & Formatting** — package manager, formatter, linter, build tool
2. **Style & Naming** — conventions specific to the language
3. **Typing / Types** — how the language expresses contracts (if applicable)
4. **Module Organization** — package/module layout, responsibility rules
5. **Error Handling** — propagation, boundaries, logging
6. **Async & Concurrency** — executor/threading rules, hot paths
7. **Verification** — exact commands: build, lint, format, typecheck
8. **Testing** — taxonomy table, file locations, commands (referencing `testing.md`)
9. **General Conventions** — constants, secrets, dependencies
```

Add a guide for a new language by copying `rust.md` or `python.md` and adapting the sections.
