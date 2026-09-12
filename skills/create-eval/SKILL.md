---
name: create-eval
description: Design and run rigorous evaluation harnesses for probabilistic, threshold-based, or LLM-judged systems. Measures failure distributions, boundary conditions, and precision/recall tradeoffs instead of shallow averages. Trigger on "build an eval for this prompt/model", "benchmark LLM outputs", "evaluate scoring model", or "design test metrics for AI pipeline".
disable-model-invocation: true
---


An eval is not asking "did this work." It's asking "where exactly does this stop working, and by how much." That is a fundamentally different task from testing, and it is the one AI agents default to doing worst — skim a few examples, produce a confident round number, move on.

This workflow does not prescribe fixed steps, because what's being evaluated varies. It prescribes a way of thinking that must hold regardless of the system.


## Why Naive Evaluation Fails (hold this in mind throughout)

- **Context exhaustion breeds superficiality.** Given hundreds or thousands of records in one prompt, a judge inspects a handful, invents a plausible-sounding aggregate, and stops.
- **Unscoped review breeds laziness.** A subagent pointed at a raw dump does a `head`/`grep`-level check, sees a few rows look fine, and assumes the rest does too.
- **Single-example threshold tweaking is close to always wrong.** Adjusting a cutoff to fix one edge case routinely breaks a much larger share of correctly-handled cases. A threshold is only touched once there is aggregated, multi-batch evidence of where signal actually separates from noise.


## Step 1 — Understand the System's Actual Execution Shape

Before designing anything, inspect how the system you're evaluating actually runs:

- What distinct stages or decision points exist?
- What is the atomic unit the system itself already processes in — a batch, a request, a session, a single item? Use that. Never invent an arbitrary evaluation chunk size that doesn't match how the system actually executes.
- What thresholds, cutoffs, or scoring boundaries govern its decisions?
- Are there multiple, structurally distinct scoring mechanisms in play (e.g. two different models measuring two different things)? If so, they must never be merged into one axis later — mark this now.

State this shape explicitly before proceeding. If it can't be stated clearly, that itself is a signal — go inspect the system more before designing the eval.


## Step 2 — Capture What Was Rejected, Not Just What Was Decided

The most dangerous blind spot in any eval is what never made it to a decision at all.

For every threshold or cutoff identified in Step 1, define what the **near-miss region** is — the band just below the cutoff — and ensure it gets logged. If you only evaluate what the system chose to act on, you can never measure what it failed to recall. This applies regardless of domain: search results just below the relevance cutoff, transactions just below a fraud score, candidates just below a similarity threshold.

Also capture, wherever applicable:
- The raw score/confidence that drove each decision, not just the final label
- An explicit reason when something was rejected — not just "rejected," but which check it failed
- Whether a candidate came from a persisted/prior source vs. the current in-flight run, if that distinction exists in the system — cold-start and steady-state behavior are usually genuinely different and shouldn't be silently blended


## Step 3 — Force Scoped, Written Judgment

Never hand a judge (LLM or subagent) the entire dataset in one pass. Split evaluation scope to match the atomic execution unit from Step 1.

For each scope:
- Give the judge the items **plus their full decision telemetry** from Step 2 — not just the raw output
- Require a written, structured report per scope — not a JSON label, not a single score. A written report forces explicitness; a label lets a skim hide.
- The judge's job per scope: identify false positives, identify likely false negatives (using the near-miss data from Step 2), and flag anything that looks like a threshold or heuristic rejected something it shouldn't have

If two structurally distinct scoring mechanisms exist (flagged in Step 1) — evaluate and report them separately. Do not let one subsystem's noise get attributed to the other.


## Step 4 — Aggregate Into Distributions, Never Anecdotes

Programmatically aggregate the per-scope telemetry into a distribution — a histogram or equivalent — across whatever confidence/score axis governs the decisions being evaluated.

- One histogram per distinct scoring mechanism identified in Step 1. Never collapse unrelated axes into a single table.
- The output should make visible the exact point where signal separates from noise — not an average, not a single number, an actual distribution across ranges.
- This distribution — not any single example — is what justifies touching a threshold. If someone (human or agent) wants to adjust a cutoff, this is the evidence they point to.


## Step 5 — Independent Synthesis

The final synthesis must be done by a separate pass from whoever did the per-scope grading in Step 3 — same reasoning as not letting a student grade their own exam.

The synthesis pass:
- Aggregates error patterns across all scopes
- Attributes every failure to a specific, named cause — never a generic "model was wrong." Causes should map back to the actual mechanisms identified in Step 1 (e.g. below-threshold, misclassification, heuristic/policy rejection, retrieval miss)
- Produces one master report: what the histograms show, what the causal breakdown of failures is, and what — if anything — is actually justified to change as a result


## Hard Rules

- Never evaluate the full dataset in a single monolithic pass
- Never adjust a threshold based on fewer than a full distribution's worth of evidence
- Never merge two structurally different scoring mechanisms onto one axis
- Never let the same pass that did per-scope grading also write the final synthesis
- If the near-miss/rejected region wasn't captured, say so explicitly — the eval is incomplete, not just conservative
