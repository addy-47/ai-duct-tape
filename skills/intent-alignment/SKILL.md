---
name: intent-alignment
description: Align on shape and process before any exploration or execution begins. Use whenever starting a new feature, workflow, or task where the process itself needs to be agreed — before reading files, before planning, before touching tools. Trigger on "let's build X", "help me do Y", "I want a workflow for Z", or any request where jumping straight to exploration would waste tokens on a shape that turns out wrong.
disable-model-invocation: true
---

# Intent Alignment

The failure this exists to prevent: exploring, planning, or building before the *shape* of the request is agreed. Shaping is cheap — it's chat. Exploration is expensive — it's tool calls. Never let the expensive thing happen before the cheap thing is confirmed.

## Step 0: Is there real ambiguity?

Judge this yourself — don't perform ambiguity that isn't there, and don't skip it when it's real.

- **Genuinely ambiguous or vague**: multiple valid interpretations of scope, a decision with real tradeoffs, a categorization or branching structure only gestured at, competing implementation paths. → Invoke **grill-me** to interrogate and settle the design tree before proceeding.
- **Clear enough to shape directly**: the goal or process is specific enough that a skeleton can be drafted and corrected in one pass. → Skip straight to Step 1.

When grill-me is invoked from here: its frontier questions may surface a need for a fact you don't have (does X exist, is Y already covered, etc.). Do not dispatch exploration for it. Mark it as an open/unknown node in the skeleton instead and resolve it during execution, after approval. The only exception is a fact already sitting in context (AGENTS.md, files already provided, things the user already told you) — reading what's already in front of you isn't exploration.

## Step 1: Produce the skeleton

Not a plan. Not a spec. A skeleton: the shape of the request or the workflow, built from what the user said (or what grill-me settled) plus whatever structure you can reasonably infer — proposing categories, gates, or sequencing is fine, that's synthesis, not exploration. What's not fine is populating it with facts you had to go look up.

- If the process is linear, the skeleton is a short numbered list of steps.
- If it branches (categorization, gates, conditional paths), draw it — boxes and arrows, however lightweight is legible.
- Mark anything unresolved plainly as unknown/pending rather than guessing or silently resolving it.
- Don't fill in real content (file names, seam names, actual categorization results) that would require reading anything. The skeleton is the shape of the work, not the work.

Keep it short. If a section of the shape is obvious, don't pad it with restated context.

## Step 2: Confirm

Present the skeleton and stop. Don't plan, don't explore, don't start step 1 of the workflow preemptively. Wait for the user to correct or approve it. Iterate here as many rounds as needed — this is still the cheap phase.

## Step 3: Execute, bounded

Once approved, the skeleton is the standing reference for the rest of the task — check items off it as they're done rather than redrafting it.

- Work one step of the skeleton at a time unless told otherwise. Finishing a step is not license to start the next one or pre-solve it "for convenience."
- Exploration is now allowed, but only in service of the *current* step. Don't read, list, or search anything the current step doesn't require.
- Baseline conduct (not asking when you should, not silently narrowing scope, defaulting to search for fast-moving ecosystem facts) is already governed by GEMINI.md — this skill doesn't restate those rules, it just gates *when* execution is allowed to start.
- If satisfying the current step would require going outside what the skeleton scoped for it, stop and ask rather than widening it quietly.

## Rules

- Never explore or touch tools before the skeleton is approved, except reading what's already in context.
- Never skip straight from request to plan or code.
- Shaping and iterating on the skeleton is free — use as many rounds as the request needs, including a full grill-me interview when warranted.
- A one-line "looks right, proceed" from the user is a complete, valid approval. Don't manufacture more ceremony than the request needs.
- If you notice yourself about to read or run something the current approved step doesn't call for — that's the stop signal, not a reason to continue since you're "already in there."
