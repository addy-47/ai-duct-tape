---
name: implement
description: Steer an agent while it writes or changes code, after a plan or spec exists or for small tasks that need none. Challenges what is being asked, checks whether it already exists, finds the smallest change in the right layer, and keeps work moving in verified steps. Use for ANY code change (feature, refactor, fix, config), even small ones, whenever you are about to write or edit code. If the cause of a bug is unknown, a test fails, or a fix attempt already failed once, switch to the rca skill.
---

Planning and spec skills decide what to build. This skill governs the doing: question the ask, build the least that fully works, in the right place, and verify each step.

The ladder in §4 is adapted from ponytail (DietrichGebert/ponytail, MIT).

## 1. Size the task first
Open with one line: `Tier: small | medium | hard, because <reason>`.
Judge by four signals: how many callers and files the change touches; whether it is reversible; how well you know this area, including whether a library or API may have changed since your training; and how fast you can observe the result.
- **Small** (local, clear, observable in seconds): read the relevant code once, change it, run the check.
- **Medium** (several files, an unfamiliar API, or behavior you can't predict by reading): trace the code paths, callers, and project conventions first. State your assumptions in two or three lines, then build in increments.
- **Hard** (unknown cause, cross-cutting, hard to reverse, data or security involved): stop building. Put the questions from §2 to the user, hand off to the alignment and planning skills, or tell the user what you need.
If the signals change mid-task (the "small" fix turns out to touch six callers), re-tier and say so.

What runs at each tier:

| Tier | §2 Challenge | §3 Scope | §4 Ladder | §5 Layer |
|---|---|---|---|---|
| Small | One-line restate, only if the ask is ambiguous | Callers and tests | Rungs 1–4 as a glance | Say which layer, one line |
| Medium | Full | Full | Full, including rung 5 if it applies | Full |
| Hard | Full, as questions to the user | Full | Do not climb until the user answers | Full |

## 2. Challenge the ask before writing
Restate the problem in one line: what outcome, for whom. Separate the requirement from the proposed solution; "add X" may be one guess at the need.
- What fails or is missing today if we do nothing?
- Is the stated solution the requirement, or one way to meet it?
- Is there a smaller change that meets the same need?
- Does it contradict an invariant, existing behavior, or an earlier decision (project rules, architecture docs)? Say so before building.

Push back with evidence (a file, a doc, observed behavior), not opinion, and keep it to one clear objection. If the user confirms after that, build what they confirm, unless it would cross a "never cut" item in §4.
A vague request ("build me X") gets the smallest version that does the core job, with what you left out named in one line. Ask a question only when the ambiguity changes the design, and ask one.

## 3. Scope: what the change must reach
List every place the change reaches: callers, tests, fixtures, config, exports, and any documentation project rules require. Check what it could break for users: data it would destroy or expose, callers that stop working. That is scope. Extra features are not.
Bug fix: grep every caller of the function you touch, then fix the root cause once, in the shared code.

## 4. Climb the ladder: take the first rung that fully works
The ladder runs after you understand the problem, not instead of it. Rungs 1–4 and 6–7 are a reflex; rung 5 is researched on purpose.
1. **Does it need to exist?** Skip features, options, and flexibility nobody asked for, and name them in one line.
2. **Already in this codebase** (a helper, component, service, pattern)? Grep before writing. Use it the way the surrounding code does.
3. **Standard library or a platform/framework feature?** Use it, unless the project has its own. A house component beats a native widget.
4. **An installed dependency?** Use it. Never add a dependency for a few lines.
5. **An external library or service?** Only when the task adds a real capability that would be substantial to write and own (parsing, codecs, protocols, crypto, model inference). Skip for fixes and small edits. Research briefly: is there a maintained option; license; recent releases and open issues; footprint (binary size, RAM, startup cost; check the project's resource budget); fit with the current stack; security. Check current docs, since training data goes stale. Give the user one or two candidates and a build-vs-buy recommendation with the reason. Adding a dependency needs the user's confirmation.
6. **Can it be one line a reader gets at a glance?** One line. A one-liner that needs decoding is not short.
7. **Otherwise:** the minimum code that works.

Rules for every rung:
- Be minimal about the solution, never about the change: finish every part the task needs, including the callers, tests, and fixtures your change breaks.
- No abstraction, wrapper, option, config, boilerplate, or "for later" code nobody asked for. Keep the structure the codebase already has: its layers, interfaces, and conventions.
- Delete only code your change makes dead, or code you added. Never remove or reduce existing behavior, validation, or error handling without showing the explicit diff and calling it out. Code you move or merge keeps its error handling and validation.
- Between options of equal size, take the one that is correct on edge cases.
- Comment only the why the code cannot show, in one line.
- New non-trivial logic (a branch, a loop, a parser, money or security, or a whole new script or app) leaves one small test or assert-based self-check. Trivial changes need none.
- A deliberate shortcut with a known limit gets a comment: `shortcut: <the limit>, <when to upgrade>`.

**Never cut, at any size:** validation at trust boundaries; error handling that prevents data loss; security; accessibility; the calibration real hardware needs; anything the user asked for; and the project's stated invariants.

## 5. Put it in the right layer
Before editing, say which layer owns the change and why it is the narrowest place for it. Check the project's layer rules (AGENTS.md, architecture docs) rather than assuming. For example, in Vox: IPC goes through the service layer, never raw `invoke` in components; settings go through the store; static copy lives in `shared/data`; the design system is consumed, not redefined; nothing heavy or lock-taking on the audio hot path. The docs are the authority; this list is a reminder.
If the right layer isn't obvious, that is a Medium signal: trace the callers first. Fix at the root once, in shared code, instead of patching each consumer.

## 6. Find out instead of guessing
When behavior depends on runtime state, timing, or an external system and reading hasn't revealed the cause, don't speculate: add a targeted log or run the reproduction. When only the user can observe it (their UI, device, credentials), ask for one specific thing: the command to run, the value to paste, the screen to check.
After one failed fix, or two focused attempts, or three tool calls with no new information, stop and follow the rca skill (probe-or-trace decision, hypotheses, circling report).

## 7. Move in verified increments
Change one logical unit, run the fastest relevant check, then continue; unverified edits don't stack. Read large sections at once, search once, then act.
- **Batch First, Compile Once:** In compiled languages (Rust, C++, Go), a "logical unit" is the complete phase or batch across its interrelated files. Never run incremental compilations per file or per line edit. Batch the full unit first, then invoke the compiler once.
- **Test Runner Economy:** Run the target test suite or binary in a single command. Never loop individual subtests sequentially.
Keep validation, error handling, security, and accessibility at any size.
If reality contradicts the plan or spec, report the mismatch before adapting.

## 8. Close out
End with one or two lines: what changed, what you skipped or did not check, and any risk the user must know. List any `shortcut:` comments you added.

## 9. Project rules
Follow task-specific requirements and repository rules. Update documentation only where project rules require it or the change materially affects it.