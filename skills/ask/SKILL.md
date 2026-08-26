---
name: ask
description: Deliver a concise, direct answer to a technical question with zero extraneous boilerplate, unrequested code blocks, or unsolicited planning steps. Trigger on direct factual queries ("how does X work", "what is the difference between A and B", "explain concept X", "quick question").
---


## Internal reasoning (do this before writing anything)

Read the question and classify it:

- **Direct** — asks for a specific fact, status, value, or yes/no
  → one or two sentences, nothing more

- **Sketch** — asks how something could look, what an approach might be, rough shape of a solution or plan
  → loose directional overview, no detail, no commitment, make clear it is a sketch not a plan

- **Explanation** — asks why something is happening, how something works, what the relationship between things is
  → enough context to actually answer, format and length determined by what the question needs

If the question is ambiguous between types — default to shorter, not longer.

## Rules (all types)
- Answer only what was asked
- Base everything on what actually exists — not what is planned or intended
- No suggestions, improvements, or next steps unless explicitly asked
- No code blocks unless a specific value or line is the answer itself
- If confidence drops at any point — flag it inline, do not paper over it
- If something is genuinely missing that blocks a confident answer — state exactly what it is and stop

## Output
Write the answer. The question determines the shape.
