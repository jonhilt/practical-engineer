---
name: slice
description: Turn the spec (and spike decisions, if any) into a concrete vertical slice design — which modules get touched, which are new, which existing capabilities are extended, and where TDD's tracer bullet will start. Use after /spec (or /spike) and before /tdd.
argument-hint: "[optional: path to the Spec]"
---

Bridge the spec's user-language description to the system's design: surface the modules and responsibilities this theory touches — which are new, which already exist and will be extended — and check them with the user before writing anything. File paths belong to TDD.

A vertical slice is the full stack a feature touches — UI, API, domain, data, external. TDD without a slice design drifts to the easiest starting point (usually a backend unit) and quietly drops the UI.

## Read the spec

Load the spec and any Technology Decisions from `/spike`. Summarise back: the headline interaction, the supporting jobs, and the tech choices.

## Explore the codebase

Build a mental model of the system's design — what modules exist, what they're responsible for, how they're meant to be used. For each supporting job, look for:

- **Existing modules and capabilities** that match the responsibility
- **Design patterns in use** — how features are organised and composed
- **Conventions to follow**

Use the Explore subagent or Grep/Glob directly. Your output is design understanding, not a file inventory.

## Plan the slice

For each supporting job:

1. **Layers touched** — UI, API, domain, data, external. Don't manufacture a concern that isn't there.
2. **Existing modules to extend** — what they already own, what this job asks of them
3. **New modules to introduce** — the responsibility each owns
4. **Dependencies** — other jobs that must exist first

**If the headline interaction involves a UI, the slice has a UI layer.** Don't let TDD quietly drop it.

### Identify the entry point

Name the outermost module the user (or caller) hits first. This is where TDD's **tracer bullet** — a lean-but-complete end-to-end slice of production code — will begin. See [../principles/tracer-bullets.md](../principles/tracer-bullets.md).

Name it by responsibility, not location. The slice commits to *which* module; TDD finds the file and decides the first test.

### Supporting jobs to cover

List each supporting job with a one-line note on which layer and module it lives in. This is *scope*, not *sequence* — TDD decides the order, and jobs aren't 1:1 with tests.

## Present the design

Show the user the modules and responsibilities: which are new, which exist and will be extended, and which module the entry point sits in. Ask: *"Does this fit how the system is organised? Am I missing anything?"*

The user knows things the files don't say. Incorporate corrections before writing.

## Write the Vertical Slice section

Append this to the spec:

<slice-template>
## Vertical Slice

**Layers touched:**
- UI: <modules, or "none">
- API: <modules, or "none">
- Domain: <modules>
- Data: <modules>
- External: <modules>

**Existing modules to extend:**
- <module>: <what it already owns, what this theory adds>

**New modules to introduce:**
- <module>: <responsibility — one thing, thin interface>

**Entry point:**
<named module the caller hits first>

**Supporting jobs (scope, not sequence):**
- <job>: <layer + module>
</slice-template>

Update spec status to `Spec ready, slice planned, awaiting /tdd`.

## What a slice plan is NOT

- **Not an architecture document.** One theory, not the whole system.
- **Not detailed design.** Interface signatures, internal data structures, and test cases belong in `/tdd`.
- **Not scope expansion.** If a supporting job is small, its slice entry should be small.
- **Not exhaustive codebase mapping.** Only modules relevant to this theory.

## Loop-backs

Go back to `/spec` if supporting jobs are too vague to map onto modules, or the system sketch assumes a data flow the codebase can't support.

Go back to `/spike` if planned modules depend on a technology choice that wasn't validated or turns out wrong.

Go back to `/theories` if the slice is too large to implement as one thin feature, or the cost is disproportionate to the theory's value.
