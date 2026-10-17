---
name: create-tickets
description: Break a plan, spec, or the current conversation into tracer-bullet tickets, each declaring what blocks it, written as one local file per ticket under .scratch. Use whenever a plan or spec needs to become agent-ready work items, or the user says "make tickets", "break this down", "split this into tasks", or "create issues from this".
---

# Create Tickets

Break a plan, spec, or conversation into **tickets**: vertical slices that each cut a thin but complete path through the system, each declaring the tickets that **block** it.

## 1. Gather context

Work from what is already in the conversation. If the user passes a reference (a spec path), read its full body and any comments.

## 2. Explore the codebase (if not already done)

Understand the current state of the code. Use the project's own vocabulary in titles and descriptions, and respect its architecture docs, decision records, and stated invariants (`AGENTS.md`).

For each candidate piece of work, apply the implement skill's ladder at planning time (§4 rungs 2–5): if an existing helper, platform feature, installed dependency, or library already covers it, shrink the ticket or drop it.

Look for prefactoring that makes the real change easy. "Make the change easy, then make the easy change."

## 3. Draft vertical slices

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer it touches (schema, API, UI, tests). It is not a horizontal slice of one layer.
- A completed slice is demoable or verifiable on its own.
- Each slice fits in a single fresh context window.
- Prefactoring comes first.

</vertical-slice-rules>

Give each ticket its **blocking edges**: the tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Spikes.** When a slice depends on something unknown (a library's behavior, a performance ceiling, a runtime condition nobody has observed), make it a `spike` ticket: it answers one question and records the finding. It ships no feature, and the build tickets that depend on the answer are blocked by it. Don't bury an unknown inside a build ticket.

**Wide refactors are the exception to vertical slicing.** A wide refactor is one mechanical change (rename a column, retype a shared symbol) whose blast radius spans the codebase, so one edit breaks many call sites at once and no vertical slice can land green. Sequence it as expand–contract:
1. **Expand:** add the new form beside the old; nothing breaks.
2. **Migrate:** move call sites over in batches sized by blast radius (per package or directory). Each batch is its own ticket, blocked by the expand. CI stays green because the old form still exists.
3. **Contract:** delete the old form once no caller remains, in a ticket blocked by every migrate batch.

If the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket. Green is promised only there.

## 4. Quiz the user

Present the breakdown as a numbered list. For each ticket show: **Title**, **Type** (build or spike), **Layer**, **Blocked by**, **What it delivers**, and **Verify**.

Ask:
- Is the granularity right (too coarse, too fine)?
- Does each ticket depend only on tickets that genuinely gate it?
- Should any tickets be merged or split?

Iterate until the user approves.

## 5. Write the tickets

Write one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). One ticket per file, never a combined file. Do not modify the source spec or plan.

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain, that means top to bottom.

<ticket-template>

# <NN>: <Ticket title>

**Type:** build | spike

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer list. For a spike: the one question it answers.

**Layer:** the layer that owns the change, in the project's own terms.

**Blocked by:** the numbers and titles of the tickets that gate this one, or "None (can start immediately)".

**Non-goals:** what this ticket must not do, in one line.

**Invariants touched:** the project invariants this ticket touches, or "None".

**Verify:** the specific command or observable outcome that proves it works.

**Status:** ready-for-agent

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

For a spike, add: **Exit:** the finding the ticket must record before it is done.

</ticket-template>

Avoid specific file paths, line numbers, or code snippets: they go stale fast. Layer names are fine. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (a state machine, reducer, schema, or type shape), inline it and note that it came from a prototype. Keep only the decision-rich parts.