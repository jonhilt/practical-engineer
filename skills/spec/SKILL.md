---
name: spec
description: Turn one theory into a clear brief — headline interaction, supporting jobs, and system sketch. The bridge between a theory and TDD. Use after /theories to specify one theory at a time.
argument-hint: "[optional: path to Theories, or paste/describe the theory]"
---

Turn one theory into a clear picture of what the software does — from the user's experience down to responsibilities and collaborations.

## Read the theory

Load it from `./theories.md` or a GitHub issue. Summarise it back — what this theory delivers, what aspect of the current state it improves, what it builds on.

## Draw out the spec

Work through three sub-questions with the user. Propose, don't prescribe.

### Headline interaction

*Ask: "What's the primary action (or actions) the user is doing here, and how do they get through it?"*

Two things to uncover, in that order: the **action(s)** that deliver the value, and the **logical interaction** by which the user gets through them. Let the user describe both in whatever shape is natural — a sentence, a few turns of dialogue, a short list. Don't impose a format.

Stay in the user's world and at the logical layer — what the user does and what the system shows back, not what it looks like or how it's built. No screens, components, modals, wireframes, or layouts. If the user drifts into physical UI ("a dropdown appears"), redirect: *"What does the user need to do at that step, logically?"* If they drift into implementation ("it calls an API"), redirect: *"That's how — what does the user experience?"*

Physical UI emerges one test case at a time in `/tdd` — don't design it here. If a mockup already exists, don't treat it as the spec: walk the user through their goal logically first, then compare against the mockup together and flag any mismatches.

### Supporting jobs

*"What must the system be able to do to deliver that experience?"*

List the capabilities that must exist to enable the headline interaction. Each job should clearly serve the headline — challenge any that don't. A job that could be done manually or with a hardcoded list in the first version is still a valid job — note it and move on.

### System sketch

*"Looking at the interaction and the jobs, how do the pieces of the system fit together? What data does each job need, and who provides it?"*

Let the user sketch the system's responsibilities and collaborations — or propose one yourself and ask them to confirm or correct. Napkin-level, not architecture, and **not** the UI's layout. If the sketch starts going into detail (interfaces, patterns, specific technologies), stop — details belong in the code. If it starts describing what the user sees (screens, components, modals), stop — that's physical UI, deferred to `/tdd`.

## Write the spec

<spec-template>
**Status:** Spec ready, awaiting /spike (or /slice if no unknowns)
**Part of:** <theories reference>
**Improves:** <aspect of current state, referencing baseline>
**Builds on:** <optional>
**Requires:** <deterministic | LLM: ... | API: ...>

## Description
<one sentence on what the software can do after this theory>

## Headline Interaction
<what the user sees and does>

## Supporting Jobs
<list of capabilities with one-line descriptions>

## System Sketch
<system responsibilities, collaborations, data flow — not UI layout>
</spec-template>

No code, test implementations, technology choices, or physical UI design (wireframes, mockups, screen layouts, component choices). The spec is a logical description; physical design emerges in `/tdd`.

## Output

Write to `./specs/<theory-number>-<slug>.md`, or edit the corresponding GitHub issue body.

## Next step

Check the **Requires** field:

- Has `LLM:` or `API:` dependencies the team hasn't used before → run `/spike`
- All deterministic or known tech → run `/slice` to map onto the codebase

## Loop-backs

Go back to `/theories` if specifying reveals the theory can't be described as a single thin slice, doesn't clearly improve any aspect of current state, or depends on capabilities that belong to a different theory.
