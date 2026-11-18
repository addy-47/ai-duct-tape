---
name: create-spec
description: >-
  Create a behavioral specification: the contracts, invariants, and design
  decisions for a capability, written before planning. Use whenever writing
  requirements, API contracts, domain behaviors, or formal specs. Trigger on
  "write a spec for", "create specification", "define behavioral spec", or
  "formalize requirements".
---

A spec is the contract a plan gets built against. It says what must be true and why, so the planner can choose the code that satisfies it. Code changes weekly; a good spec survives renames, refactors, and minor version bumps unchanged.

## Step 1: Find the lane
Infer the kind of spec (behavior, architecture, interface, data, or another) and state it in one line. Stay in that lane. A concern that belongs to another lane becomes its own spec; say so. Ask one question only if the answer would change the lane.

## Step 2: Decide what earns a place
Test every line: if every function were renamed and every dependency took a minor version bump, would this line still be true? If yes, it is spec. If no, it is plan or code, and the planner owns it.

- **Name the decisions that shape the design**: concurrency primitive, ownership pattern (single writer, actor and channel), failure-isolation mechanism, a critical library when it is the decision. Give the reason in a line.
- **Describe shapes, don't transcribe them.** For an enum, say what each member means and when it occurs. For a struct, say what it carries and which invariants bind it. The planner picks names and types.
- **Anchor behavior to user stories** with stable IDs. The plan later links each function to the story it serves, so the spec needs no function names. Point at existing code by its role or component, not its symbols.

## Step 3: Make it checkable
Cover what applies, in the order that fits this spec:
- **Intent and scope**: the problem, who it serves, what is out of scope.
- **User stories**: "As a…, I want…, so that…", with IDs. Cover failure and edge paths too. IDs are never renumbered or reused.
- **Behavior rules**: one observable statement each, strength marked MUST/SHOULD/MAY. Write event, state, and failure rules as "When/While/If… the system shall…". State preconditions, postconditions, and invariants where a contract has them.
- **Vocabulary**: closed sets and key terms, as in Step 2.
- **Decisions**: choice, reason, rejected alternative.
- **Verification**: how each behavior is observed from outside. Tests target observable behavior, not internals.
- **Open questions**: mark unresolved items as open; never fill them with guesses.

## Step 4: Draw when shape beats sentences
When the content is about order or relationships, a diagram carries it better than prose: who talks to whom and in what order (sequence), states something moves through (state machine), who owns what and where data flows (flow/ownership). Pick the diagram type from the content, use a text-based format so it diffs in the file, and keep prose for the rules a diagram can't hold (guards, failure paths). If nothing in the spec has that shape, draw nothing.

## Step 5: Voice
Direct and active. Every sentence states something a test could confirm or a decision someone could reverse. Concrete conditions, not "should be secure". The reader is the planner and the engineer; a non-developer should still follow the stories and rules.

## Step 6: Save and gate
Save to `specs/[name]-spec.md`, present it, and wait for approval.
A spec is living: when requirements change, update the spec first, then re-evaluate everything built against it. It never drifts silently from what was built.
