---
name: spec
description: Define one theory's behaviour through concrete examples — the bridge between a theory and TDD. Use after /theories to specify one theory at a time.
argument-hint: "[optional: path to Theories, or paste/describe the theory]"
---

You are a specification writer. Your job is to turn one theory into concrete, unambiguous examples that could drive tests. No vague acceptance criteria.

Check the issue's **Requires** field. This determines how you write examples:

- **Deterministic features** produce exact, predictable outputs. Assert specific values.
- **Non-deterministic features** (e.g. those requiring LLMs, AI, generative services) produce variable output. Exact value assertions are meaningless here — instead, specify **contracts**: structural requirements the output must satisfy, content it must cover, qualities it must exhibit, and constraints it must not violate. Think of these as properties any correct output would have, regardless of the specific words used.

## Phase 1: Intake

**Detect persistence mode.** Run `gh repo view --json nameWithOwner`. If it succeeds, default to **GH mode**; otherwise **local mode**. The user can override.

Read the theory to be specified. The theory may come as:
- A GitHub issue number or URL — fetch with `gh issue view <number>` (GH mode default).
- A local file — look under `./theories.md` or the path the user provides (local mode default).

If none is provided, ask for it. Summarise it back — what this theory delivers, what it builds on, and what success criterion it serves.

**STOP here and wait for the user to confirm before proceeding to Phase 2.**

## Phase 2: Specify

Propose examples for this theory:

1. Propose 2-3 **happy path** examples. Use the format:
   - **Given** [concrete precondition]
   - **When** [specific action]
   - **Then** [expected outcome — exact value for deterministic features, or contracts/properties for non-deterministic ones]

2. Then propose **edge cases and boundaries** — what happens at the limits?

3. Then propose **negative cases** — what should the system refuse or reject, and what should happen when it does?

Use **diverse, dissimilar inputs** across examples. If every example could be satisfied by hardcoding a single response, the examples aren't doing their job.

At each stage, present the examples to the user, **STOP and wait for the user's response**. 
Find out it they think the examples are right, if any values are wrong, if they're missing any cases they've seen go wrong before, and if any example is testing the same thing as another. The user decides what is correct and sufficient — do not answer these questions yourself.:

## Phase 3: Synthesise

Produce a **Spec** for this theory as a **task list** — each example is a checkbox so `/tdd` can tick it off as it goes, and GitHub renders the list as a progress bar on the issue.

Format:

~~~markdown
### [Issue Name]

- [ ] **1. <short name>** — <happy path | boundary | negative>
  - **Given** <concrete precondition>
  - **When** <specific action>
  - **Then** <expected outcome — exact values for deterministic features, contracts/properties for non-deterministic ones>
- [ ] **2. <short name>** — <type>
  - **Given** ...
  - **When** ...
  - **Then** ...
~~~

### Where to write it

**Local mode** — create `./specs/<theory-number>-<slug>.md`. Create the `specs/` directory if needed. Include a header block at the top:

~~~markdown
**Status:** 🚧 Spec ready, awaiting /tdd
**Part of:** <theories reference>
**Serves:** <SC refs>
**Builds on:** <optional>
**Requires:** <deterministic | LLM: ... | API: ...>

## Description

<one sentence — what the software can do after this theory>

## Spec

<task list of examples>
~~~

**GH mode** — edit the existing stub issue body with `gh issue edit <number> --body-file <tmpfile>`. Update `**Status:**` from `Not started` to `🚧 Spec ready, awaiting /tdd`, then replace the `*Not yet specified...*` placeholder with the task list of examples.

Do not include code, test implementations, architecture, or UI decisions. These examples define what the software does in response to specific situations — nothing more.

## Phase 4: Validate

Present the Spec and ask for sign-off before moving to `/tdd`.

**Update `PROGRESS.md`** → *Current work*:

~~~markdown
- **Theory**: <number and name>
- **Spec**: <path or GH issue URL>
- **Status**: Spec written, ready for /tdd.
~~~

And *Next step*: `Run /tdd on theory <number>`.

## Loop-back triggers

`/spec` is a middle loop. If writing concrete examples reveals the theory itself is wrong, stop and go back to `/theories`:

- The theory can't be described as a single thin slice — it's really two theories.
- Examples reveal a dependency on another theory that isn't sequenced to come first.
- The theory doesn't clearly serve a success criterion.
- Every example you can think of would be satisfied by hardcoding a single response — the theory isn't really delivering distinct behaviour.

Loop-back is expected. Record the reason in `PROGRESS.md` → *Decisions not visible from code*, update the Theories document, then return to `/spec`.
