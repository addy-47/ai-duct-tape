# AGENTS.md

Navigation map for any agent working in or consuming this repo. This repo is a **source of truth library** — rules, roles, skills, and code style guides that get copied into projects. Nothing here is specific to one product.

## Layout

```
rules/          Global rules — defaults that apply to every project, every thread.
roles/          Role definitions — agent personas (backend, frontend, architect, ML, QA, test).
skills/         Workflows — named procedures referenced as /<skill-name>.
style-guides/   Per-language engineering standards + a language-agnostic general guide.
THIRD_PARTY.md  Attribution for vendored/referenced external skills.
```

## What to Read When

| Task | Read |
|---|---|
| Any task, always | `rules/global-rules.md` |
| Writing/modifying backend code | `rules/global-rules.md` + `roles/backend-engineer.md` + `style-guides/general.md` + the language guide (`style-guides/rust.md` / `go.md` / `python.md`) |
| Writing/modifying frontend code | `rules/global-rules.md` + `roles/frontend-engineer.md` + `style-guides/general.md` + `style-guides/typescript-react.md` |
| Architectural decisions / new features | `rules/global-rules.md` + `roles/system-architect.md` + skills: `intent-alignment.md`, `architect.md`, `create-spec.md`, `validate.md` |
| Planning implementation | skills: `create-plan.md`, `create-loop.md`, `modify-plan.md` |
| Testing | `roles/test-engineer.md` + skills: `test-plan.md` |
| Verifying/reviewing evidence | `roles/qa-engineer.md` + skills: `review.md`, `feedback-review.md` |
| Debugging a regression | skills: `rca.md` |
| ML model/data work | `roles/ml-research-engineer.md` + skills: `create-dataset.md`, `create-eval.md` |
| Refactoring | skills: `refactor-clean.md` (behavior-preserving) or `refactor-arch.md` (behavior-changing) |
| Emergency fix | skills: `hotfix.md` |
| Stress-testing a plan/idea | skills: `grill-me.md` |
| Ending a thread / new thread | skills: `handoff.md` |
| Frontend polish pass | role `frontend-engineer.md` + `impeccable` (see THIRD_PARTY.md) |

## Resolving Skill References

A `/name` or `name` reference in any file resolves to `skills/<name>.md`. All cross-references in this repo must resolve to a real file — if a reference points nowhere, that is a bug in the repo.

## Consuming This Repo (for projects)

This repo is not meant to be depended on in place. **Copy relevant files per project** — see `README.md` for per-project-type copy recipes. After copying:

- `roles/*.md` → the project's `.agents/rules/` (or equivalent per host)
- `skills/*.md` → the project's skills directory per host (`.agents/skills/`, `.opencode/skills/`, `~/.config/opencode/skills/`, etc.)
- `style-guides/*.md` → a project docs dir (e.g. `.agents/rules/` alongside roles)
- `rules/global-rules.md` → global agent config or project rules, per host

## Host-Specific Content

Anything marked ⚠️ in this repo (currently: `agy-subagent`, `/schedule`) is Antigravity-specific and should be filtered out when copying to other hosts.
