---
name: create-sprints
description: Apply a simple, well-defined rule or change across a large number of files or instances. Not for complex or architecturally significant work — for tedious, mechanical breadth. Prevents premature "done" declarations via a persistent checklist and sprint batching.
---

This is for tasks that are simple in nature but large in surface area — apply this style rule everywhere, enforce this line cap across every function, remove this pattern from every file in a directory. The task itself requires no clever judgment per instance. The risk is entirely that the agent checks a handful of cases, sees the pattern holds, and declares the whole thing done.

**Sampling is not completion.** A task like this is only done when every single instance in the full scope has actually been addressed and checked off — not when a representative sample looks fine.

---

## Step 1 — Enumerate the Full Scope, Deterministically

Before touching anything, establish the exact, complete scope using a command that cannot lie by omission — not by reasoning from memory or a partial look at the directory.

- Use `grep`, `find`, `ls -R`, or equivalent to produce the exact list of files/functions/instances this applies to
- Get an exact count. State it explicitly: "N files identified, M total instances to address."
- If the rule applies conditionally (e.g. "every function over 50 lines," not every function) — run the actual check that determines which instances qualify. Don't estimate.

## Step 2 — Build a Persistent Checklist

Create a checklist artifact — a file, not just in-context tracking — listing every single unit identified in Step 1, unchecked.

```
- [ ] path/to/file_a.rs
- [ ] path/to/file_b.rs
- [ ] path/to/file_c.rs
...
```

This file is the actual source of truth for progress — not what the agent remembers doing. It must be updated as each unit is completed, and it must be re-readable at any point (including in a fresh thread) to know exactly what remains.

## Step 3 — Divide Into Sprints

Break the full checklist into sprints — batches of a manageable size (judge based on file size and complexity, not a fixed number). A sprint is a unit of *volume*, not a unit of *dependency* — sprints don't depend on each other's outcome the way phases do. They exist purely to keep each individual pass focused and verifiable rather than attempting the entire scope in one unchecked sweep.

If the volume is large enough to benefit from it, use a sub-agent per sprint — same agent instance for the duration of that sprint so context and pattern consistency hold, fresh sub-agent for the next sprint if starting clean is preferable given the volume.

## Step 4 — Execute One Sprint at a Time

For each file/instance in the current sprint:
- Apply the rule
- Verify the applied change actually satisfies the rule — don't just apply and assume, check the specific condition (line count, absence of the flagged pattern, whatever the rule actually is)
- Check it off in the checklist file immediately — not at the end of the sprint, per item

If a specific instance doesn't cleanly fit the rule (genuine edge case, not laziness) — flag it explicitly in the checklist with a note, do not silently skip it, and do not let one ambiguous case stall the whole sprint.

## Step 5 — After Each Sprint

Report:
- How many items completed this sprint
- How many remain, referencing the checklist directly
- Any flagged edge cases needing a decision

Continue to the next sprint without waiting for approval, unless a flagged edge case needs a decision first — this is meant to run through tedious volume efficiently, not stop for permission at every batch. Stop and ask only when something genuinely doesn't fit the rule as stated.

## Step 6 — Done Means the Checklist Says So

The task is not complete until every item in the checklist from Step 2 is checked off or explicitly flagged and resolved. Before declaring done:

- Re-run the Step 1 enumeration command again to confirm no instances were missed or newly introduced
- Confirm the checklist shows 100% resolution
- Report the final count: total addressed, total flagged, total remaining (should be zero)

Never declare a sprint task complete based on "I did a good chunk of it and it's working" — completion is binary against the checklist, not a vibe.
