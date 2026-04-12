---
name: spec
description: Turn one theory into a clear brief — headline interaction, supporting jobs, and napkin sketch. The bridge between a theory and TDD. Use after /theories to specify one theory at a time.
argument-hint: "[optional: path to Theories, or paste/describe the theory]"
---

Turn one theory into a clear picture of what the software does — from the user's experience down to responsibilities and collaborations.

## Read the theory

Load it from `./theories.md` or a GitHub issue. Summarise it back — what this theory delivers, what aspect of the current state it improves, what it builds on.

## Draw out the spec

Work through three sub-questions with the user. Propose, don't prescribe.

### Headline interaction

*"When this theory is working, what does the user actually see and do?"*

Describe the single experience that proves this theory delivers value. Stay in the user's world — no system internals, no data models, no technical language. If the user starts describing implementation ("it calls an API"), redirect: *"That's how — what does the user experience?"*

### Supporting jobs

*"What must the system be able to do to deliver that experience?"*

List the capabilities that must exist to enable the headline interaction. Each job should clearly serve the headline — challenge any that don't. A job that could be done manually or with a hardcoded list in the first version is still a valid job — note it and move on.

### Napkin sketch

*"How do these jobs fit together? What data does each one need, and who provides it?"*

Sketch responsibilities and collaborations — napkin-level, not architecture. If the sketch starts going into detail (interfaces, patterns, specific technologies), stop — details belong in the code.

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

## Napkin Sketch
<responsibilities, collaborations, data flow>
</spec-template>

No code, test implementations, or technology choices.

## Output

Write to `./specs/<theory-number>-<slug>.md`, or edit the corresponding GitHub issue body.

## Next step

Check the **Requires** field:

- Has `LLM:` or `API:` dependencies the team hasn't used before → run `/spike`
- All deterministic or known tech → run `/slice` to map onto the codebase

## Loop-backs

Go back to `/theories` if specifying reveals the theory can't be described as a single thin slice, doesn't clearly improve any aspect of current state, or depends on capabilities that belong to a different theory.
