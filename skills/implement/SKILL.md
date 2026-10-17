---
name: implement
description: Steer an agent while it writes or fixes code, after a plan or spec exists or for small tasks that need none. Sizes the task, decides when to reason, when to add logs and reproduce, and when to ask the user to look, and keeps work moving in verified steps. Use for any code change, bug fix, or debugging session.
---

Planning and spec skills decide what to build. This skill governs the doing: spend effort where it pays off, and get real evidence fast instead of reasoning to a confident guess.

## 1. Size the task first
Open with one line: `Tier: small | medium | hard, because <reason>`.
Judge by four signals: how many callers and files the change touches; whether it is reversible; how well you know this area, including whether a library or API may have changed since your training; and how fast you can observe the result.
- **Small** (local, clear, observable in seconds): read the relevant code once, change it, run the check.
- **Medium** (several files, an unfamiliar API, or behavior you can't predict by reading): trace the code paths, callers, and project conventions first. State your assumptions in two or three lines, then build in increments.
- **Hard** (unknown cause, cross-cutting, hard to reverse, data or security involved): stop building. Hand off to the alignment and planning skills, or tell the user what you need.
If the signals change mid-task (the "small" fix turns out to touch six callers), re-tier and say so.

## 2. Choose how to find out: probe, think, or ask
Compare the cost of observing with the cost of being wrong.
- **Probe** (add targeted logs, run the reproduction, print the real value) when behavior depends on runtime state, timing, concurrency, or an external system; when reading hasn't revealed the cause; or when observing takes seconds and more reasoning takes minutes.
- **Think and trace** when the logic is deterministic, the path is short and readable, or probing would be destructive or expensive.
- **Ask the user** when only they can observe it (their UI, device, environment, credentials), when the answer is taste or product intent, or when a quick visible draft settles it faster than speculation. Ask for one specific thing: the command to run, the value to paste, the screen to check.

## 3. Treat every fix as a hypothesis
Before applying a fix you can't verify by reading, state the hypothesis and the observation that would confirm it. Reproduce first, and confirm the fix with the same reproduction.
After one failed fix, or two focused attempts, stop reasoning and instrument. Put probes at boundaries (what goes into and comes out of each step) so one run shows where reality departs from your model. Remove the probes when done.

## 4. Move in verified increments
Change one logical unit, run the fastest relevant check, then continue; unverified edits don't stack. Read large sections at once, search once, then act.
Build the minimum that meets the requirement, following the global minimize order, and keep validation, error handling, security, and accessibility at any size.
If reality contradicts the plan or spec, report the mismatch before adapting.

## 5. Notice when you are circling
Three tool calls with no new information, or the same area edited three times: stop and report **Known** (confirmed), **Unknown**, and **Next probe** or your question for the user.

## 6. Project rules
Follow task-specific requirements and repository rules. Update documentation only where project rules require it or the change materially affects it.
