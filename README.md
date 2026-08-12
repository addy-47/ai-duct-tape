# AI Duct Tape

A **source of truth library** of agent rules, roles, skills, and code style guides. Nothing here is tied to a specific product or stack — files are generic by design and get **copied into projects**, not depended on in place.

This repo is what the owner reaches for when setting up a new project's agent context: the global rules that should govern every thread, the role personas that define what each agent does (and doesn't) own, the reusable workflows (`/skill-name`), and the per-language style guides.

## Contents

| Directory | What's in it |
|---|---|
| `rules/` | `global-rules.md` — defaults for every project, every thread, every stack |
| `roles/` | Agent personas: backend-engineer, frontend-engineer, system-architect, ml-research-engineer, qa-engineer, test-engineer |
| `skills/` | Workflows referenced as `/<name>`: planning, refactoring, testing, RCA, hotfix, handoff, review, grilling, ML/data, and more |
| `style-guides/` | `general.md` (stack-agnostic) + `rust.md`, `typescript-react.md`, `python.md`, `go.md` |
| `THIRD_PARTY.md` | Attribution for vendored/referenced external skills |

**Agents navigating this repo:** read `AGENTS.md` — it maps task types to the files you should read.

## Copy Recipes

Copy the relevant files into each project. Do not clone or submodule this repo into a project.

### Full-stack project

```
rules/global-rules.md            → .agents/rules/ or global agent config
roles/*.md                       → .agents/rules/
skills/<name>/SKILL.md            → .agents/skills/  (per host, see below)
style-guides/general.md          → .agents/rules/
style-guides/typescript-react.md → .agents/rules/
style-guides/rust.md             → .agents/rules/   (if backend is Rust)
```

### Backend-only project

```
rules/global-rules.md
roles/backend-engineer.md  roles/system-architect.md
roles/qa-engineer.md       roles/test-engineer.md
style-guides/general.md  style-guides/<language>.md
```

### ML / data project

```
rules/global-rules.md
roles/ml-research-engineer.md  roles/qa-engineer.md  roles/test-engineer.md
skills/create-dataset/SKILL.md  skills/create-eval/SKILL.md  skills/feedback-review/SKILL.md
```

### Where skills/roles land per host

| Host | Roles | Skills |
|---|---|---|
| Antigravity | `.agents/rules/` | `.agents/skills/` (⚠️ filter host-specific content) |
| opencode (project) | `.opencode/` or `.agents/rules/` | `.opencode/skills/` |
| opencode (global) | `~/.config/opencode/` | `~/.config/opencode/skills/` |
| Claude Code | `.claude/` | `.claude/skills/` |

## Host-Specific Content

Two items are Antigravity-specific and should be **filtered out** when copying to other hosts:
- `skills/agy-subagent/SKILL.md` — the `agy` CLI subagent orchestration
- `/schedule` in `roles/ml-research-engineer.md` — Antigravity's native cron command

Both are flagged ⚠️ in-place; `THIRD_PARTY.md` has details.

## Contributing / Extending

- **New skill:** add `skills/<name>/SKILL.md` with a `name:` and `description:` frontmatter block. Any `/name` reference in another file must resolve to it.
- **New role:** add `roles/<role>.md` following the existing persona skeleton (how you think → invariants → skills you reach for → what you don't own → boundary-leak detection).
- **New language guide:** copy `style-guides/rust.md`, adapt the sections (see `style-guides/README.md`).
- **Vendored external skills:** record attribution in `THIRD_PARTY.md`.
