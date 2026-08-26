---
name: report
description: Produce a comprehensive, evidence-based markdown report artifact analyzing system states, architecture audits, or complex technical questions. Trigger whenever the user asks for a "formal report", "written audit", "comprehensive analysis document", or "in-depth system assessment" as an artifact rather than a quick chat reply.
---


## Rules

- Always present the report as a artifact not as a response.
- Answer only the question asked
- Base everything on what actually exists and happens — not what the plan says, not what should happen
- Depth and format are yours to decide based on what the question actually needs
- No suggestions, no improvements, no next steps unless explicitly asked

## Internal Reasoning (do this before writing your answer)

- What exactly is being asked?
- What is the actual source of truth for this? (code, docs, research, MCP, etc.)
- Trace the real behavior end to end — not the intended behavior
- Where does your confidence drop? Flag those points explicitly in your answer
- What is the minimum structure this answer needs to be clear?

## Output

Write your answer. Let the question determine the shape.
If confidence drops at any point, flag it inline — do not paper over uncertainty.
