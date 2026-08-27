# Roles Library

A collection of **7 specialized agent personas** that establish distinct operational mindsets, strict invariant boundaries, and task ownership for AI coding assistants.

---

## 🎭 Available Roles

| Role File | Domain | Primary Mindset & Invariants |
|---|---|---|
| [backend-engineer.md](backend-engineer.md) | Backend & APIs | Systems-first, fault-tolerant, data-boundary validation, strict connection pooling, transactions for multi-table ops, zero leaked stack traces. |
| [frontend-engineer.md](frontend-engineer.md) | Frontend & UI | State-first, UI-second. Zero perceptual lag, closed design-system enforcement, strict service boundaries, mandatory 5 core UI states, usability over flair. |
| [system-architect.md](system-architect.md) | Architecture & Specs | Long-term maintainability, trade-off analysis, explicit contract definition, boundary isolation, avoiding premature optimization. |
| [ml-research-engineer.md](ml-research-engineer.md) | ML, Data & Evals | Scientific rigor, deterministic data pipelines, leak-free splits, audited datasets, adversarial evaluation design over shallow LLM summaries. |
| [qa-engineer.md](qa-engineer.md) | Verification & Audit | Adversarial verification, evidence-based review, finding false positives, verifying real runtime behavior over static assumptions. |
| [test-engineer.md](test-engineer.md) | Automated Testing & Evals | Comprehensive test taxonomy, `/create-test` seam tracing, `/test` execution loops, `/mutate` empirical regression validation, zero unverified green tests. |
| [idea-validator.md](idea-validator.md) | Idea Validation | Idea validation & feasibility analysis | 

---

## 🏛️ Persona Anatomy

Each role in this directory follows a consistent behavioral structure:
1. **How You Think:** The foundational mental model and decision priorities for that discipline.
2. **Invariants:** Non-negotiable rules that must never be broken regardless of messy existing code.
3. **Before You Commit to a Direction:** The specific skills (`/intent-alignment`, `/validate`, `/review`, etc.) the persona invokes before acting.
4. **What This Role Does NOT Own:** Clear domain boundaries preventing the persona from overstepping into other layers.
5. **Boundary-Leak Detection:** Self-check triggers alerting the user if the agent starts performing tasks belonging to another role.

---

## 📦 How to Use Roles

Copy the relevant role files into your project's agent rules directory (e.g. `.agents/rules/` or `.opencode/`). 

You can activate a role manually in your prompt or configure an orchestration skill (like `/agy-subagent`) to initialize a subagent with a dedicated role persona.
