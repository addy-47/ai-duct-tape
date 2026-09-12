---
name: create-plan
description: Turn one or more specs into an ordered, executable plan against the real codebase — not a narrative document, not a per-spec breakdown. Front-loads a relentless clarification pass, then sequences work by actual blast radius and dependency, sized so a single agent can execute each batch without fail. Use at the start of implementation or when the user says "create a plan", "how should we implement this", "plan the rollout", or "break this into batches".
---

## What planning is not

Planning is not reading every file the spec touches, reading every function in those files, and cataloguing what each one currently does. That's documentation, and it doesn't scale — five specs against a 30k-line codebase will not fit in anyone's head or context that way, and most of what gets read is never load-bearing to the plan.

Reading code during planning has exactly one purpose: finding the real, current consumers of the specific things about to change. That's targeted tracing of a symbol or interface, not exploration of the codebase. If a step in this process starts to look like "open every file the spec mentions and summarize it," stop — that's the failure mode this skill exists to prevent.

## No backward compatibility

Never plan a compatibility bridge, a dual-write window, or a deprecated-but-still-supported path. When a shared interface changes, every current consumer of it is updated in the same batch, atomically. There is no "old and new coexist for a while." This is what makes blast radius — not which spec a change came from, not a fixed vertical slice — the thing that actually determines how work groups together.

---

## Step 1 — Grill relentlessly, before any sequencing starts

Read the specs against the actual codebase, not against each other in isolation. For every place a spec is silent, ambiguous, conflicts with what the code currently does, or requires a judgment call the spec never states — check the codebase first. Plenty of apparent gaps are already answered by something that exists (a convention, an existing pattern, a value already defined elsewhere) and don't need a question at all. Only what's left after that — genuine judgment calls the code can't settle on its own — goes to the user. Do not record it as an "assumption" and move on; assumptions surviving into the plan is the single biggest source of wasted work, more than any implementation mistake downstream. If an answer opens a new ambiguity, ask again. Keep going until nothing is left unresolved. Nothing in the steps below starts until this is done — the highest-friction translation is spec-to-plan, not plan-to-code, so this is where the effort belongs.

## Step 2 — Extract atomic changes and find real blast radius

Flatten every spec into a list of discrete, concrete changes — a function whose behavior changes, a module that's new, a call site that needs updating, a schema field added or removed. Tag each with the real file and symbol it touches. Once extracted, which spec a change came from stops mattering — don't carry that grouping forward.

For each item, trace its actual current consumers in the codebase: who calls this, who depends on this struct, who reads this table. This is the targeted read from the section above — follow the specific thing changing, not the surrounding code. Trace one hop by default: direct callers and direct consumers only. Follow a consumer further only if it re-exposes the changed thing as part of its own external contract (it becomes a new thing something else depends on) — don't chase transitively just because the graph keeps going; that's how a targeted trace turns into exploration again. The result is a real blast radius per change, not an assumed one.

Once every item has been traced, check the list back against the specs it came from: does every requirement, edge case, and constraint that was in the original specs now appear somewhere on this list? Anything that got lost in the flattening gets added back or explicitly logged as intentionally dropped — never silently orphaned.

## Step 3 — Cluster by shared blast radius

Group changes that have to move together because they share an interface, struct, schema, or piece of state with no bridge between its old and new form. Two changes from unrelated specs that touch the same shared thing belong in the same batch. Two changes from the same spec that share no blast radius and don't depend on each other can be split apart — being in the same document doesn't make them the same unit of work.

## Step 4 — Order by real dependency

Two kinds of dependency matter, and they're easy to conflate: a change can be **structurally** blocked (it literally can't be written until something else exists — a new variant, a new field, a new module) or **verification**-blocked (it can be written early, but can't be confirmed correct until something else exists or is reproducible). Order batches so nothing is scheduled before what it structurally needs. Batches with no dependency between them can run in either order.

## Step 5 — Size each batch to what one agent can execute in one sitting, without fail

The ceiling on batch size is execution reliability, not build health. A batch that changes a shared struct or migrates a schema across many files may not build green until every file in it is done — that's expected for genuinely atomic changes, not a sign the batch is broken, and it must be marked as such so whoever executes it doesn't mistake mid-batch red for failure. Other batches are naturally small and should stay green the whole way through — mark those too, so it's clear what "in progress but fine" looks like versus what "in progress and broken" looks like for each batch.

## Before presenting: check the plan against itself

Once the batching is done, check it rather than declaring it finished. Did a compatibility bridge slip in anywhere despite the rule against them? Does every requirement from the specs actually land in a batch, per the check at the end of Step 2? Are the dependency edges in Step 4 actually respected in the batch order, not just noted? Can each batch be verified as done on its own terms, not just assumed to work because it followed logically? Fix what fails this before showing the plan, not after.

---

## Two required outputs

Both are fluid in structure — use whatever headings and layout fit the work. What matters is what each one contains.

**An implementation plan document**, which must contain:
- The target end-state, stated plainly.
- Every grill-me question raised and its resolved answer — by the time this exists, there should be no open assumptions left to flag.
- The blast-radius and dependency reasoning that produced the batching — enough that someone reading it can see *why* a boundary exists between two batches, not just that one does.
- For each batch: what's in it, what it depends on, whether it's expected to hold a green build throughout or only once it's fully complete, and what "done" actually means for it.
- Risks, and anything that had to be inferred rather than being stated outright anywhere in the specs.
- Confirmation that every requirement from the specs was accounted for somewhere in the batching, and a note on anything deliberately dropped and why.

**A checklist file, and only a checklist** — no narrative, no rationale, no exploration notes; all of that lives in the plan document instead. It must contain:
- Every file touched, organized under the batch it belongs to.
- Every function or symbol changed, with one line on what changes about it.
- What prior batch(es) each batch depends on.

## Resolution tapers with distance

Batches that will be executed soon get full file-and-symbol detail in both documents. Batches further downstream can stay coarser — their real shape depends on what the earlier batches actually produce once they're built, and over-specifying them now against a codebase that hasn't changed yet just produces detail that gets thrown away and re-derived later against reality instead of guesswork.
