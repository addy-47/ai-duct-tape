# Third-Party Skills & Attributions

This repo vendors or references third-party agent skills. Attribution and upstream links live here so nothing is silently claimed as original.

| Skill | Status | Upstream | License |
|---|---|---|---|
| `skills/grill-me.md` | **Vendored** (content copied into this repo) | [mattpocock/skills](https://github.com/mattpocock/skills) — `skills/productivity/grilling/SKILL.md` | MIT |
| `impeccable` (referenced in `roles/frontend-engineer.md`) | **Referenced** — install separately; commonly available as a global skill | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | — |
| `skills/agy-subagent.md` | **Vendored** (host-specific) | Originates from the user's Antigravity IDE setup; `agy` CLI | — |

## Host-specific notes

- **`agy-subagent`** only applies inside the Antigravity IDE (uses the `agy` CLI, disk-persisted conversations, `~/.gemini/antigravity-cli/`). On other hosts (opencode, Claude Code, Copilot) use the host's native subagent support instead — the skill documents this itself.
- **`/schedule`** (referenced in `roles/ml-research-engineer.md`) is an Antigravity-native cron command only. It is intentionally *not* vendored or referenced as a skill here; the role file keeps only the host-agnostic "scheduled check-ins" discipline.

## Updating vendored skills

`skills/grill-me.md` is a copy. To refresh it: fetch the upstream file, re-apply the frontmatter style used in this repo, and update the version/path note above.
