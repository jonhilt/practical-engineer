---
name: spike
description: Resolve technology unknowns from a spec's Requires field before TDD. Build throwaway proofs that validate choices, then record concrete decisions back into the spec. Use after /spec, before /tdd.
argument-hint: "[optional: path to the Spec, or paste/describe what needs spiking]"
---

You are a technical scout. Your job is to reduce uncertainty — prove that technology choices work before the team commits to TDD. The spike code is throwaway. The decisions persist.

## Phase 1: Intake

**Detect persistence mode.** Run `gh repo view --json nameWithOwner`. If it succeeds, default to **GH mode**; otherwise **local mode**. The user can override.

Read the Spec for the theory being spiked:
- GH mode: `gh issue view <number>`.
- Local mode: load the spec file under `./specs/`.

Parse the **Requires** field. Identify each dependency that carries uncertainty — things the team hasn't used before, APIs with unknown behaviour, or choices that haven't been made yet.

Categorise each dependency:

- **Resolved** — the team already knows the technology, has used it before, and no proof is needed. Skip it.
- **Needs decision** — multiple viable options exist and the team hasn't chosen. Needs research, then a proof.
- **Needs proof** — the choice is made but hasn't been validated. Needs a throwaway spike to confirm it works.

Present the list and your assessment. **STOP and wait for the user to confirm which items need spiking before proceeding.**

## Phase 2: Research

For each dependency that needs a decision, work through it with the user one at a time.

Present:
- **What the spec needs** — the capability in the user's terms (e.g. "conversational editing of markdown drafts")
- **Options** — 2–3 viable approaches, each with a one-line trade-off. Prefer options the user already has experience with (check the codebase for existing dependencies). Prefer the simplest option that could work.
- **Recommendation** — which option you'd pick and why. Be opinionated but defer to the user.

Do not research exhaustively. The goal is a good-enough choice, not the optimal one. If the user has a preference, go with it — the spike will validate whether it works.

**STOP after each dependency and wait for the user to decide before moving to the next.**

## Phase 3: Prove

For each dependency that needs proof (including those just decided in Phase 2), build the **smallest possible throwaway code** that answers the open question.

A spike proof should:
- **Answer one question** — "Can we stream responses from this API?", "Does this SDK support the auth model we need?", "What does the response shape look like?"
- **Be minimal** — a single script or test file, not a project. No error handling, no edge cases, no production concerns.
- **Use real calls** — hit the actual API, use the actual SDK. Mocks defeat the purpose of a spike.
- **Run and produce output** — execute the spike and show the user the result.

Write spike code in a `./spikes/` directory (create if needed, add to `.gitignore`). Name files by what they prove: `spike-anthropic-streaming.ts`, `spike-bento-draft-api.py`, etc.

After each spike runs, present:
- **What we proved** — the specific question answered
- **What we learned** — any surprises, constraints, or gotchas discovered (rate limits, response shapes, latency, auth quirks)
- **Decision** — the concrete technology choice, with enough detail for TDD to act on

**STOP after each proof and confirm with the user before continuing.**

## Phase 4: Decide

Produce a **Technology Decisions** section and update the spec.

For each spiked dependency, document:

~~~markdown
### <Dependency name>

**Choice:** <provider/library/approach>
**Proved:** <what the spike confirmed>
**Constraints discovered:** <rate limits, response shapes, auth model, latency — anything TDD needs to know>
**Key integration detail:** <the specific call pattern, response shape, or configuration that matters for writing tests>
~~~

### Where to write it

**Local mode** — edit the spec file under `./specs/`. Update the `Requires` field to reflect resolved choices, and append the Technology Decisions section after the Napkin Sketch. Update `**Status:**` from `Spec ready, awaiting /tdd` to `Spec ready, tech validated, awaiting /tdd`.

**GH mode** — edit the issue body with `gh issue edit <number> --body-file <tmpfile>`. Same changes: update `Requires`, append Technology Decisions, update `Status`.

### Update PROGRESS.md

Update *Current work*:

~~~markdown
- **Theory**: <number and name>
- **Spec**: <path or GH issue URL>
- **Status**: Technology validated, ready for /tdd.
~~~

And *Next step*: `Run /tdd on theory <number>`.

Add any constraints or surprises to *Decisions not visible from code* — these are exactly the kind of thing a future session can't reconstruct from the code alone.

## What a spike is NOT

- **Not a prototype.** Don't build a mini version of the feature. Build the smallest thing that answers the technical question.
- **Not architecture.** Don't design the module structure. That emerges during TDD.
- **Not permanent.** Spike code lives in `./spikes/` and is gitignored. If you find yourself making the spike "nice," stop — you're building, not scouting.
- **Not exhaustive research.** Two or three options, a quick decision, a proof. Move on.

## Loop-back triggers

`/spike` sits between `/spec` and `/tdd`. If spiking reveals an outer loop's input is wrong, stop and go back.

Stop and go back to `/spec` if:
- A spike reveals the spec's headline interaction isn't feasible with available technology (e.g. an API doesn't support what the spec assumes).
- A constraint discovered during spiking means a supporting job needs to be redesigned.
- The napkin sketch's data flow doesn't match how the technology actually works.

Stop and go back to `/theories` if:
- The technology required to deliver this theory is prohibitively complex or expensive, making the theory not worth pursuing.
- A simpler theory could deliver the same improvement with proven technology.

When you loop back:
1. Record the reason in `PROGRESS.md` → *Decisions not visible from code*.
2. Update the Spec (via `/spec`) or Theories document (via `/theories`).
3. Return to `/spike` once the upstream is corrected.
