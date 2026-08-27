# AGENTS.md

Navigation map for any agent working in or consuming this repo. This repo is a **source of truth library** — rules, roles, skills, and code style guides that get copied into projects. Nothing here is specific to one product.

## Layout

```
rules/          Global rules — defaults that apply to every project, every thread.
roles/          Role definitions — agent personas (backend, frontend, architect, ML, QA, test).
skills/         Workflows — named procedures referenced as /<skill-name>.
style-guides/   Per-language & cross-cutting engineering standards (general, testing, design, rust, ts, etc.).
THIRD_PARTY.md  Attribution for vendored/referenced external skills.
```

## What to Read When

| Task | Read |
|---|---|
| Any task, always | `rules/global-rules.md` |
| Writing/modifying backend code | `rules/global-rules.md` + `roles/backend-engineer.md` + `style-guides/general.md` + the language guide (`style-guides/rust.md` / `go.md` / `python.md`) |
| Writing/modifying frontend code | `rules/global-rules.md` + `roles/frontend-engineer.md` + `style-guides/general.md` + `style-guides/design.md` + `style-guides/typescript-react.md` |
| Architectural decisions / new features | `rules/global-rules.md` + `roles/system-architect.md` + skills: `intent-alignment`, `architect`, `create-spec`, `validate` |
| Planning implementation | skills: `create-plan`, `create-loop`, `modify-plan` |
| Testing, evals & mutation validation | `roles/test-engineer.md` + `style-guides/testing.md` + skills: `create-test`, `test`, `mutate` |
| Verifying/reviewing evidence | `roles/qa-engineer.md` + skills: `review`, `feedback-review` |
| Debugging a regression | skills: `rca` |
| ML model/data work | `roles/ml-research-engineer.md` + skills: `create-dataset`, `create-eval` |
| Refactoring & mechanical breadth | skills: `refactor-clean` (behavior-preserving), `refactor-arch` (behavior-changing), `create-sprints` (sprint batching) |
| Emergency fix | skills: `hotfix` |
| Stress-testing a plan/idea | skills: `grill-me` |
| Ending a thread / new thread | skills: `handoff` |
| Frontend polish pass | role `frontend-engineer.md` + `impeccable` (see THIRD_PARTY.md) |

## Resolving Skill References

A `/name` or `name` reference in any file resolves to `skills/<name>/SKILL.md`. All cross-references in this repo must resolve to a real file — if a reference points nowhere, that is a bug in the repo.

## Consuming This Repo (for projects)

This repo is not meant to be depended on in place. **Copy relevant files per project** — see `README.md` for per-project-type copy recipes. After copying:

- `roles/*.md` → the project's `.agents/rules/` (or equivalent per host)
- `skills/<name>/SKILL.md` → the project's skills directory per host (`.agents/skills/`, `.opencode/skills/`, `~/.config/opencode/skills/`, etc.)
- `style-guides/*.md` → a project docs dir (e.g. `.agents/rules/` alongside roles)
- `rules/global-rules.md` → global agent config or project rules, per host

## Host-Specific Content

Anything marked ⚠️ in this repo (currently: `agy-subagent`, `/schedule`) is Antigravity-specific and should be filtered out when copying to other hosts.
