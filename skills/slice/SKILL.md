---
name: slice
description: Turn the spec (and spike decisions, if any) into a concrete vertical slice plan — which modules get touched, which are new, which existing code is modified, and where TDD's tracer bullet will start. Use after /spec (or /spike) and before /tdd.
argument-hint: "[optional: path to the Spec]"
---

Bridge the spec's user-language description to the codebase's module structure. The system sketch says *"something renders the editor"* — your job is to say *"the `<WriteScreen>` component at `apps/web/src/features/write/WriteScreen.tsx` renders it."*

A vertical slice is the full stack a feature touches — UI, API, domain, data, external. TDD without a slice plan drifts to the easiest starting point (usually a backend unit) and quietly drops the UI.

## Read the spec

Load the spec and any Technology Decisions from `/spike`. Summarise back: the headline interaction, the supporting jobs, and the tech choices.

## Explore the codebase

Build a mental model of what already exists — real exploration, not guessing. For each supporting job, look for:

- **Existing modules** that match the responsibility
- **Codebase conventions** — monorepo layout, feature folders, layering patterns, how new features are typically added
- **Frontend layer** — framework, feature folder pattern, how screens handle state and API calls
- **Data layer** — where entities live, persistence approach

Use the Explore subagent or Grep/Glob directly. Be specific about files you looked at and patterns you found. The user knows things about the codebase you can't read from files — present findings and invite correction.

## Plan the slice

Walk through each supporting job one at a time. For each, answer:

1. **Which layers does it touch?** UI, API, domain, data, external. If a job only touches backend, say so — don't manufacture a UI concern.
2. **Existing modules to modify** — concrete file paths and what changes
3. **New modules to create** — concrete file paths and what responsibility they own
4. **Dependencies** — other jobs that must exist first

**If the headline interaction involves a UI, the slice has a UI layer.** Don't let TDD quietly drop the UI by omitting it here.

### Identify the entry point

Name the outermost component, endpoint, or function the user (or caller) hits first when exercising the headline interaction. This is where TDD's **tracer bullet** — a lean-but-complete end-to-end slice of production code — will begin. See [../principles/tracer-bullets.md](../principles/tracer-bullets.md).

- **UI feature** — the top-level screen or component (e.g. `<WriteScreen>` at `apps/web/src/features/write/`)
- **API-only feature** — the handler or endpoint
- **Domain library** — the public entry function

The slice commits to *where* the tracer bullet begins. TDD decides what the first test looks like when it gets there.

### Supporting jobs to cover

List each supporting job with a one-line note on roughly where it lives in the module map (layer + file if known). This is *scope*, not *sequence* — TDD decides the order based on what each red-green cycle teaches it, and jobs are not 1:1 with tests. One behavioural test may cover several jobs; some jobs may only be exercised through outer tests and never earn their own.

## Write the Vertical Slice section

Append this to the spec:

<slice-template>
## Vertical Slice

**Layers touched:**
- UI: <components, or "none">
- API: <endpoints/handlers, or "none">
- Domain: <entities/aggregates/services>
- Data: <storage/persistence>
- External: <APIs/services>

**Existing modules to modify:**
- `<path>`: <what changes and why>

**New modules to create:**
- `<path>`: <responsibility>

**Conventions followed:**
- <e.g. feature folder at `apps/web/src/features/write/`, minimal API endpoint pattern>

**Entry point:**
<outermost component/endpoint/function the user or caller hits first — e.g. `<WriteScreen>` at `apps/web/src/features/write/`>

**Supporting jobs to cover (scope, not sequence):**
- <job>: <one-line note on roughly where it lives in the module map>
- <job>: ...
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
