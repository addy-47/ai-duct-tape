---
name: create-loop
description: Build a loop outline/skeleton from a goal — structure only, no implementation detail. Defines layers (goals), phases inside each layer, persistent sub-agents, gates, and scheduled check-ins. Depth scales with input complexity. /create-plan fills in each phase once reached.
---


This is not a plan and not a spec. A plan details *how* to do one unit of work. A spec defines *what must be true*. This defines the *shape of the run* — the goals in sequence, the phases inside each goal, where a sub-agent must sign off before continuing, where the human must sign off before the next goal starts, and how the loop is checked on while running unattended.

Output of this workflow is a artifact `implementation_plan.md` file. It gets detailed phase-by-phase using `create-plan` skill  as the loop reaches each one — same rolling-detail principle as implementation plans. Never front-load detail into every phase here.


## Step 1 — Read the Goal

Input can vary wildly in depth:
- A 3-4 line goal stated directly, mid-project
- A pointer to a `specs/` directory for a new build
- A mix — a stated goal plus existing docs/specs as grounding

Read whatever is given. If specs exist, they are the source of truth for what "correct" means at any gate that checks compliance.

## Step 2 — Assess Complexity (determines skeleton depth)

Before building the outline, judge:
- How many genuinely distinct **layers** (small goals) does this overall goal break into?
- Within each layer, how many **phases** are actually needed — what does each phase produce that the next phase depends on?
- Where does a **sub-agent** need to grade something the main agent shouldn't grade itself on?
- Where does a **human** need to verify before the next layer starts?
- What's the realistic unattended run length? Short goals may need zero scheduled check-ins. Long ones need periodic ones.

State this assessment briefly before writing the skeleton. Simple goals get simple skeletons — do not manufacture layers, phases, or gates that aren't earned by actual complexity.


## Step 3 — Build the Skeleton

### Overall Goal
The full goal, restated precisely. This is what every layer ultimately serves.

### Layers
A layer is a small goal — a meaningful unit of the overall goal, not a pipeline stage.

For each layer, only:
- **Layer N — [name]**
- The specific goal this layer achieves
- What it hands to the next layer once complete

Do not detail phases here beyond naming them — depth comes from `/create-plan` when execution reaches that phase.

### Phases (inside each layer)
Phases are sequential steps toward that layer's goal. Each phase's scope depends on what the previous phase actually produced — not planned in isolation.

For each phase, only:
- **Phase N — [name]**
- One-line purpose
- Type: `research` / `implementation` / `data operation` / `sub-agent review` / `test pass` / conditional
- What it depends on from the previous phase
- Skip condition if any
- Sub-agent gate if this phase requires one (name which sub-agent, see below)

### Sub-Agents (persistent — critical)
For every sub-agent referenced across any layer or phase:

- **Name / role** — e.g. "QA sub-agent", "data quality sub-agent", "e2e schema sub-agent"
- **This is a single persistent identity for the entire loop.** Every phase that calls on this role sends a new task to the *same* sub-agent thread — never spin up a fresh instance. Prior findings, prior context, and prior rejections must carry forward.
- What it is grading against (specs, schema, prior layer's output — be specific)
- What it must report back (pass/fail + detailed findings, not just a verdict)
- Whether its verdict feeds a sub-agent gate

### Sub-Agent Gates (mid-layer, between phases)
A sub-agent gate is a hard stop inside a layer — a named sub-agent must return pass before the next phase in that layer proceeds.

For each:
- **Gate** — which sub-agent, exact pass condition
- On pass: which phase resumes
- On fail: which phase the layer repeats from — always explicit, never "try again"

### HITL Gates (end of layer)
A HITL gate sits at the end of a layer, not mid-phase. The layer's output is presented to the human for manual verification before the next layer begins.

- **Gate** — what the human is verifying, what "approved" means
- On approval: next layer begins
- On rejection: which phase or layer to repeat from, with what changed

### Scheduled Check-Ins
Only if the run is long enough to warrant unattended monitoring. Define as a recurring native check-in (cron-style), not a one-off:

- **Interval** — e.g. every 15 minutes
- **Prompt injected at each check-in** — should be adversarial, not a status ping. Push back by default:
  - Has every issue actually been looked at, or assumed fixed?
  - Is the current phase detailed enough, or is it coasting on a vague scope? If vague — stop and force it through `/modify-plan` before continuing.
  - Is progress real, or is the loop repeating the same failed attempt without a new hypothesis?
- Check-ins never approve — they only interrogate. Sub-agent gates and HITL gates are the only things that approve progression.


## Step 4 — Present and Hand Off

Present the full skeleton. Wait for approval.

Once approved:
- Layer 1, Phase 1 gets detailed via `/create-plan`, using this skeleton's definition as its scope
- As the loop reaches each subsequent phase, `/create-plan` or `/modify-plan` details it based on what the previous phase actually produced — never plan phases far ahead in detail
- Sub-agent gate failures trigger `/modify-plan` on the phase being repeated, incorporating what the failing sub-agent's report actually found — not a blind retry
- Sub-agents persist across the entire loop by name — do not re-launch a role fresh unless the skeleton explicitly retires it
