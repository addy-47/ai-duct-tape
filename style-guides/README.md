# Style Guides

Per-language engineering standards. **Agents doing write operations must read the relevant guide before modifying code.**

## Guides

| File | Scope |
|---|---|
| `general.md` | Language-agnostic best practices — applies to every stack: modularity, API standardization, CLI discipline, security, Docker, `.gitignore`. Read this **always**. |
| `rust.md` | Rust backend/services code |
| `typescript-react.md` | TypeScript / React frontend code |
| `python.md` | Python services and scripts |
| `go.md` | Go services and APIs |

## How to Use

1. Always read `general.md` — it applies regardless of language.
2. Read the language guide for the stack you're writing.
3. Guides are meant to be **copied into a consuming project** (e.g. `.agents/rules/` or a project `docs/` dir) — see the repo root `README.md` for copy recipes.

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
8. **Testing** — taxonomy table, file locations, commands
9. **General Conventions** — constants, secrets, dependencies
```

Add a guide for a new language by copying `rust.md` and adapting the sections.
