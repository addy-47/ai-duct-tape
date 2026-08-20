# Global Rules

This directory contains the foundational, non-negotiable governance rules that apply to every project, every agent thread, and every tech stack.

---

## 📜 Core Rule: `global-rules.md`

[global-rules.md](global-rules.md) defines **Rule Zero** and critical operational constraints designed to eliminate common AI failure modes (hallucinations, silent assumptions, hidden bugs, and stale internal knowledge).

### Key Invariants Summary

1. **Ask, Don't Assume (Rule Zero):**
   * If there is more than one valid interpretation, stop and ask.
   * If a missing requirement changes scope or approach, route the ask properly (`/intent-alignment` for product outcomes, `/grill-me` for design decisions, or direct chat for inline clarity).
2. **Blockers Are Reported, Never Worked Around:**
   * Never substitute fake mock data, skip failing tests, or present partial results as complete.
   * **The 2-Attempt Rule:** If an error persists after 2 targeted fix attempts with no new hypothesis, stop and escalate to the user with exact evidence.
3. **No Silent Simplification:**
   * Never narrow scope or drop edge cases mid-task without explicit approval.
   * Prefer surgical, precise edits over sweeping rewrites.
4. **Treat Ecosystem Knowledge as Stale by Default:**
   * SDK versions, model endpoints, and CLI tools move fast. Always search first and trust explicitly provided user versions over internal training defaults.
5. **Standardized Output Categorization:**
   * Every non-skill response concludes with:
     * 🐛 **Bug:** High-risk issues, flawed assumptions, or misleading conclusions.
     * ⚖️ **Trade-off:** Meaningful pros and cons of architectural choices.
     * 💡 **Improvement:** High-value optimizations supported by sound engineering principles.
6. **Quick Sanity Check:**
   * Run the stack's fast build/syntax check (`pnpm build`, `cargo check`, etc.) before declaring any task done. Never use `npm` when `pnpm` is standard.

---

## 📦 How to Apply

Copy `global-rules.md` into your agent host's global configuration or project rule directory:
* **Antigravity / OpenCode:** `.agents/rules/global-rules.md`
* **Global OpenCode:** `~/.config/opencode/rules/global-rules.md`
* **Claude Code:** `.claude/CLAUDE.md` or global system prompt
