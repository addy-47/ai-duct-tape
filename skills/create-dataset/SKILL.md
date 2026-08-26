---
name: create-dataset
description: Generate, audit, and quality-gate ground-truth datasets for model fine-tuning or evaluation. Implements deduplication, schema validation, and multi-layer micro-batch pipelines to prevent poisoned weights. Trigger on "build a training dataset", "generate ground truth pairs", "create eval dataset", or "curate fine-tuning data".
---


A dataset used to fine-tune a model is not like normal output. Bad code fails loudly — a bad crash, a failed test, a broken build. A bad label in a training pair fails silently. It gets baked into weights and surfaces weeks later as a model that's subtly wrong in ways nobody can trace back to its source. Treat every pair as something that must be correct, not something that looks plausible.

Use `/grill-me` before generation begins if there is any ambiguity in the taxonomy, labels, domain boundaries, or what "correct" means for a pair. Do not proceed on assumption — an ambiguous label definition corrupts every pair generated under it.


## Step 1 — Assess Actual Complexity (determines pipeline depth)

Before building anything, determine what this dataset actually needs. Do not default to a full multi-layer pipeline if it isn't earned.

Ask:
- How many pairs/examples are needed, roughly?
- How many categories, labels, or classes exist, and how ambiguous is the boundary between them?
- Is there a directional or structural policy that must be enforced (e.g. only certain category pairs are valid, some combinations are forbidden)?
- What does this dataset feed — a full fine-tune, a lightweight classifier, an eval set? Higher downstream stakes justify more audit layers.
- Is human-in-the-loop review feasible at this scale, or must this be fully automated end to end?

**Small, low-ambiguity dataset** (few hundred pairs, few labels, clear boundaries) → single generation pass + one verification pass is enough. Do not build a 4-layer pipeline for this.

**Large or high-ambiguity dataset** (thousands of pairs, many labels, structural policy to enforce, feeding a real fine-tune) → full micro-batch pipeline below is justified.

State this assessment explicitly before proceeding.


## Step 2 — Define Ground Truth Before Generating Anything

This must exist and be confirmed correct before a single pair is generated:

**Label/Category Definitions**
- Exact definition of every category or domain involved
- Exact definition of every label/edge/class that can connect them — what it means, not just its name
- Explicit positive and negative examples for each label

**Structural Policy (if applicable)**
- Which category-to-category connections are valid at all
- Explicit statement of what is forbidden — reverse or invalid combinations are a common and serious failure mode if not enforced
- The threshold or filter (if any) used to pre-screen candidates before generation

**What "Correct" Means for a Single Pair**
- The exact criteria a pair must satisfy to be considered ground truth
- What makes a pair ambiguous or invalid, so those can be rejected rather than forced into a label

If any of this is unclear or you are inferring definitions rather than being given them — stop and use `/grill-me`. Do not generate against an assumed definition.


## Step 3 — Generation Approach

**Determinism where it matters:**
- Initial candidate generation can use some sampling variance if it helps diversity
- Verification and audit passes must use zero-temperature deterministic evaluation with strict equality against the expected label — not "close enough"

**Full RAG/definitional context in every prompt:**
Every generation and verification prompt carries the complete definitions from Step 2 — category definitions, label semantics, policy rules, and explicit examples. A prompt that assumes the model already knows the taxonomy produces ambiguous or invented labels.

**Batch size (for large datasets):**
Split generation into independently-gated batches rather than one monolithic run. If a batch fails its quality gate, halt before it corrupts anything downstream — do not let a bad batch propagate into the master dataset.


## Step 4 — Audit Layers (apply what's earned by Step 1's assessment)

For datasets that justify the full pipeline, structure audit as independent layers — each is a hard gate, not a soft check:

**Deterministic/Schema Audit**
- Uniqueness — no duplicate IDs, no duplicate pairs
- Structural policy compliance — zero violations of the valid-connection rules from Step 2
- This layer is mechanical and cheap — run it first, before spending compute on model-based audit

**Independent Model-Based Audit**
- A separate judge pass (different from the generator) evaluates candidates at zero temperature against the ground-truth criteria
- Define the hard pass threshold explicitly before running it — e.g. "batch must score ≥ 90% agreement to proceed." Do not decide the bar after seeing the result.
- Audit the full batch, not a sample, if the dataset is small enough to afford it. For very large datasets, a defined and justified sampling rate is acceptable — state what it is and why.

**On Gate Failure**
- Halt. Do not proceed to commit the batch.
- Report exactly which pairs failed and why
- Determine whether the issue is in the generation prompt, the label definitions, or genuinely ambiguous source data — fix at the right level, don't just regenerate blindly


## Step 5 — Commitment

- Only pairs that passed every applicable gate are committed to the master dataset
- Commitment is atomic per batch — a batch either fully commits or fully doesn't
- Deduplicate against already-committed data before final commit, not just within the new batch
- Log the audit result per batch — pass rate, failure reasons, what was excluded — this is part of the dataset's provenance, not throwaway output


## Step 6 — Final Report

- Total pairs generated vs. committed vs. rejected
- Audit scores per batch (if batched)
- Any structural policy violations caught and corrected
- Duplicate rate
- Anything that required a definition change mid-run — flag clearly, since this may mean earlier batches need re-auditing against the corrected definition

Do not declare the dataset ready for fine-tuning until every batch has cleared its gates and the report is reviewed.
