---
description: Architect a system, service, or feature through iterative discussion before anything is formalized. Produces HLD or LLD depending on what's asked. Entry point after intent is aligned, before a spec is written. Trigger with "design a system for", "how should we architect", "let's think through the design for", "architect this before we spec it".
---

This is a discussion, not a document generator. The point is to think through the design together — propose, get pushed back on, revise — until the shape is right. Only then does it get formalized into a spec via `/create-spec`.

Do not produce a final polished design on the first pass and call it done. Treat every output here as a draft to be argued with.

---

## Step 0 — Scale and Intent (mandatory, first)

Before designing anything, establish what this actually needs to hold up under. If not already clear:

- Is this a proof of concept, an MVP, or something expected to run at real production scale?
- Roughly what load — a handful of users, thousands, or millions?
- What's the actual timeline — days, weeks, or is this meant to last?

State the calibration explicitly: "Designing this as [POC/MVP/production] — everything below is scoped accordingly."

This determines everything that follows. A POC does not get failover, horizontal scaling, or caching layers unless there's a specific reason it needs them now. Do not default to production patterns out of habit.

---

## Step 1 — Determine HLD or LLD

Infer from what's being asked, state which one you're doing:

- **HLD** — components, boundaries, data flow, ownership, how pieces talk to each other. No endpoint-level detail, no schema-level detail.
- **LLD** — inside a component already agreed at HLD level: API contracts, data model, specific endpoints, request/response shape, error handling.

If the request is genuinely asking for both, do HLD first, get alignment, then move to LLD. Never jump straight to LLD on something that hasn't had its boundaries agreed yet.

---

## Step 2 — Check for Existing Ground Truth

Before proposing anything:
- Does a spec already exist in `specs/` for this area? If so, the design must respect it or explicitly flag where it needs to change and why.
- Does existing architecture documentation already define adjacent boundaries this needs to fit into?
- Read what exists. Do not redesign something already decided without flagging it first.

---

## Step 3 — Ask Before Proposing

If the request is underspecified, ask before designing — don't fill gaps with assumptions:
- What does this actually need to do, in plain terms?
- What's explicitly out of scope?
- Is there an existing pattern elsewhere in the system this should be consistent with?

Keep this to the minimum needed to start a real proposal. Not an interrogation — just enough to not waste a round trip.

---

## Step 4 — Propose

Produce a draft, scoped to what Step 0 and Step 1 established:

**HLD draft:**
- Components and what each owns
- Data flow between them (diagram as ASCII or clearly described)
- Where state lives
- What's explicitly not being solved here

**LLD draft:**
- Data model
- API contract — endpoints, request/response shape, error cases
- What existing contract this must remain compatible with, if any

**Every non-trivial decision gets a one-line reason.** Not a full trade-off writeup here — if a decision is genuinely contested, flag it and use `/validate` to go deep on that specific point rather than resolving it inline.

State explicitly what you'd revisit if scale or requirements change later.

---

## Step 5 — Iterate

Present the draft and stop. Wait for pushback.

When feedback comes back:
- Update only what changed — don't regenerate the whole design from scratch each round
- If a decision keeps getting revisited without new information, say so — that's a sign the actual constraint hasn't been stated yet, go back to Step 3

Repeat until the person confirms the shape is right. Don't rush to "final" — this is expected to take a few passes.

---

## Step 6 — Handoff

Once aligned, state clearly: this design is ready to be formalized.

Do not write the spec yourself here. Point to `/create-spec` — that workflow takes this aligned design and formalizes it into the language-agnostic behavioral contract. This workflow's job ends at alignment, not at the artifact.