# Skills Library

A collection of **23 structured agent workflows** designed for autonomous pair programming and AI agent orchestration.

A skill in this repo is an executable standard operating procedure (SOP) that guides an AI agent through a specific engineering discipline — complete with discovery steps, human-in-the-loop checkpoints, and strict verification gates.

---

## ⚡ Quick Decision Matrix

| If your goal is to... | Run Skill |
|---|---|
| Figure out what we are actually building before writing code | `/intent-alignment` |
| Pressure-test a new product idea, feature, or tech choice | `/idea-validator` or `/grill-me` |
| Validate a proposed tech stack swap, dependency, or architectural pivot | `/validate` |
| Draft a high-level (HLD) or low-level (LLD) system architecture | `/architect` |
| Create a language-agnostic behavioral specification | `/create-spec` |
| Break architecture down into a step-by-step phased execution plan | `/create-plan` |
| Adjust an existing implementation plan mid-thread | `/modify-plan` |
| Build an autonomous loop skeleton with multi-phase agent checkpoints | `/create-loop` |
| Implement a thin end-to-end slice across all layers before going deep | `/build-vertical` |
| Audit a plan or bug report against the actual code before touching anything | `/feedback-review` |
| Perform an adversarial senior code review for scale and maintainability | `/review` |
| Run an iterative testing loop until code genuinely passes | `/test-plan` |
| Clean up / decouple code with zero logic or behavior changes | `/refactor-clean` |
| Re-architect subsystem logic while intentionally superseding old behavior | `/refactor-arch` |
| Apply an emergency surgical patch to a broken production system | `/hotfix` |
| Investigate the root cause of a regression or unexpected bug | `/rca` |
| Generate a curated, gated ground-truth dataset for fine-tuning/eval | `/create-dataset` |
| Design a rigorous evaluation framework for probabilistic/scored AI systems | `/create-eval` |
| Write a comprehensive evidence-based report or state analysis artifact | `/report` |
| Get a concise, direct answer to a question without code or plans | `/ask` |
| Summarize context and produce a structured handover for the next thread | `/handoff` |
| Spin up isolated, persona-based subagents in Antigravity | `/agy-subagent` ⚠️ |

---

## 🧭 Skills by Engineering Phase

### 1. 🎯 Discovery & Alignment
* [skills/intent-alignment/SKILL.md](intent-alignment/SKILL.md) — Aligns mental models on user outcomes and constraints before any planning or code begins.
* [skills/idea-validator/SKILL.md](idea-validator/SKILL.md) — Socratic scrutiny of brand-new ideas, features, or product proposals before investing engineering time.
* [skills/grill-me/SKILL.md](grill-me/SKILL.md) — Relentlessly grills the user about design decisions, tradeoffs, and edge cases.
* [skills/validate/SKILL.md](validate/SKILL.md) — Deep, evidence-backed validation of proposed architectural changes or new dependencies.

### 2. 🏗️ Architecture & Planning
* [skills/architect/SKILL.md](architect/SKILL.md) — Iterative system design producing High-Level (HLD) or Low-Level (LLD) architecture docs.
* [skills/create-spec/SKILL.md](create-spec/SKILL.md) — Language-agnostic behavioral specification (defines *what* must be true, never *how*).
* [skills/create-plan/SKILL.md](create-plan/SKILL.md) — Phased implementation planning (Phase 1 planned in detail; later phases kept high-level).
* [skills/modify-plan/SKILL.md](modify-plan/SKILL.md) — Mid-thread adaptation of active implementation plans in response to discoveries.
* [skills/create-loop/SKILL.md](create-loop/SKILL.md) — Autonomous loop skeleton with layers, sub-agents, gates, and scheduled check-ins.
* [skills/build-vertical/SKILL.md](build-vertical/SKILL.md) — Vertical-slice execution methodology: builds a thin end-to-end trace across all layers first.

### 3. 🔬 Review & Verification
* [skills/feedback-review/SKILL.md](feedback-review/SKILL.md) — Pre-implementation audit: checks proposals against real codebase reality to eliminate hallucinations.
* [skills/review/SKILL.md](review/SKILL.md) — Adversarial senior code review hunting for complexity, scale breaks, and anti-patterns with drop-in replacements.
* [skills/test-plan/SKILL.md](test-plan/SKILL.md) — Generic testing workflow looping until tests are genuinely passing with valid assertions.

### 4. 🛠️ Execution & Refactoring
* [skills/refactor-clean/SKILL.md](refactor-clean/SKILL.md) — Behavior-preserving refactor (cleanup, modularization, dead code removal, zero logic changes).
* [skills/refactor-arch/SKILL.md](refactor-arch/SKILL.md) — Architecture-replacing refactor (new patterns and data flows superseding legacy logic).
* [skills/hotfix/SKILL.md](hotfix/SKILL.md) — Abbreviated, surgical emergency fix for broken systems with mandatory rollback steps.

### 5. 🧠 Machine Learning & Evaluation
* [skills/create-dataset/SKILL.md](create-dataset/SKILL.md) — Ground-truth dataset creation with auditing, deduplication, and quality gating.
* [skills/create-eval/SKILL.md](create-eval/SKILL.md) — Robust evaluation harness design for probabilistic systems, LLM outputs, and scoring classifiers.

### 6. 🔍 Analysis, Ops & Handoff
* [skills/rca/SKILL.md](rca/SKILL.md) — Investigative root-cause analysis for regressions and outages.
* [skills/report/SKILL.md](report/SKILL.md) — Generates formal written report artifacts with evidence and data flow analysis.
* [skills/ask/SKILL.md](ask/SKILL.md) — Direct, unbloated answers with zero boilerplate or unwanted code sketches.
* [skills/handoff/SKILL.md](handoff/SKILL.md) — End-of-thread context consolidation and structured prompt for resuming in a fresh session.
* [skills/agy-subagent/SKILL.md](agy-subagent/SKILL.md) ⚠️ — Antigravity CLI orchestration for launching dedicated subagents in separate sessions.

---

## 📦 How to Copy Skills into Projects

Skills are formatted according to the universal Agent Skills standard: a directory named after the skill containing a `SKILL.md` with YAML frontmatter.

| Agent Host | Target Destination |
|---|---|
| **Antigravity** | `.agents/skills/<skill-name>/SKILL.md` |
| **OpenCode (Project)** | `.opencode/skills/<skill-name>/SKILL.md` |
| **OpenCode (Global)** | `~/.config/opencode/skills/<skill-name>/SKILL.md` |
| **Claude Code** | `.claude/skills/<skill-name>/SKILL.md` |

*(Note: Items marked with ⚠️ are Antigravity-specific and should be omitted on other hosts.)*
