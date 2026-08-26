---
name: intent-alignment
description: Align on what we are about to build before planning, architecture, or implementation begins. Use whenever starting a new feature, complex task, or when requirements are ambiguous, open-ended, or underspecified. Trigger on phrases like "let's build X", "I want to create", "help me add", or whenever clarifying the true problem and success criteria is needed before writing code or specs.
---


## Workflow

1. Restate the request in plain English.

   * Explain it like a CEO or PM would.
   * Focus on outcome, not implementation.

2. Surface assumptions.

   * What is being assumed?
   * What is unclear?
   * What could be interpreted multiple ways?

3. Ask only the minimum questions required to remove ambiguity.

   * No technical questions unless they change scope or outcome.
   * No planning yet.

4. Produce a concise alignment summary.

Format:

What we're building:

* ...

Not in scope:

* ...

Success looks like:

* ...

5. Get confirmation.

Only after alignment is confirmed may planning begin.

## Rules

* Never jump from request to plan.
* Never hide assumptions inside a plan.
* Never discuss architecture before intent is clear.
* Keep summaries short.
* If the request is already clear, skip questions and provide the alignment summary immediately.

## Optional

If requested, expand the alignment summary into a formal SPEC.md before planning.
