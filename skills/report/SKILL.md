---
name: report
description: Produce an evidence-based markdown report artifact analyzing system states, architecture, incidents, or comparisons. Trigger on "formal report", "written audit", "comprehensive analysis document", "in-depth system assessment" as an artifact rather than a chat reply.
---

# Report

Always an artifact, not a chat response. Answer only the question asked. Base everything on what actually exists and happens — not what the plan says, not what's supposed to happen.

## Step 1 — Classify the report type

Pick the closest fit. This decides the shape — don't default to prose-with-headers for everything.

| Type | When | Shape |
|---|---|---|
| **Status** | "Where does X stand right now" | Current state by component/area, what's done, what's not, blockers. No history, no recommendations. |
| **Audit** | "Does X actually do/follow what it's supposed to" | Claim vs. reality, one row per claim checked. Pass/fail/partial per item, evidence cited. |
| **Investigative** | "Why/how does X happen" (not a live incident — use `rca` for that) | Trace from question to evidence to answer. Confidence flagged inline wherever it drops. |
| **Comparative** | "X vs Y", "which approach/option" | Shared criteria, evaluated side by side. No recommendation unless explicitly asked. |
| **Architecture** | "Document/assess how X is built" | Structure first (layers/components/data flow), then notable behavior or gaps. |

If the request doesn't cleanly fit one type, say so in one line and state which you're defaulting to and why — don't silently pick one.

## Step 2 — Internal reasoning (before writing)

- What exactly is being asked, and which type from Step 1 does it match?
- What's the actual source of truth — code, docs, logs, a tool, direct observation? Not the plan, not the spec, not what should be true.
- Trace real behavior end to end.
- Where does confidence drop? Mark those points now so they don't get smoothed over while writing.

## Step 3 — Write

- Structure follows the type from Step 1 — don't force sections that type doesn't call for (an Audit doesn't need a narrative trace; a Status report doesn't need a verdict).
- Flag confidence drops inline, where they occur — not as a disclaimer at the end.
- No suggestions, improvements, or next steps unless explicitly asked.
- Length matches what the question needs, not a target page count.
