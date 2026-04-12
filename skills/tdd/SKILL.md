---
name: tdd
description: Implement one slice through strict outside-in TDD. Review any findings from previous cycles, then fire a tracer bullet end-to-end (first slice only) or loop Red/Green/Refactor through the slice's behaviours. Use after /slice, one slice at a time.
argument-hint: "path to the slice file (e.g. specs/blog/slices/01-create-draft.md)"
---

Implement **one slice** through strict outside-in TDD. `/slice` has already done the planning — modules, responsibilities, behaviours, concrete examples. This skill executes that plan for one slice at a time.

Every test verifies **behaviour**, not implementation. Behaviour is what the software does from the outside — inputs, outputs, observable effects through the public interface. Implementation is how it does it — internal structure, private state, which collaborator was called.

A mortgage calculator test asserts *"£200,000 at 5% over 25 years produces £1,169.18 monthly"* — stable across any refactoring of the internals. **Do not write tests that are coupled to implementation.**

## Read the slice

Load the slice file passed as argument. It carries:

- **Modules** — entry point, new modules with their responsibilities, existing modules being extended, external boundaries
- **Behaviours** — each tagged `validation: automated | contract | human`, with Given/When/Then or contract assertions
- **Seeds** — fixture inputs for contract assertions
- **Frontmatter** — `tracer: true|false`, `depends_on`, current `status`

Verify every slice listed in `depends_on` is `status: done`. If not, stop — slices must run in dependency order.

## Review findings

Before writing any test, open `specs/<theory>/findings.md`. If any entries are unreviewed, present them to the user and wait for input. Three outcomes:

1. **Not a real issue** — mark the entry reviewed (move under `## Resolved`), proceed.
2. **User amends a pending slice file by hand** — mark the entry reviewed, proceed with the (possibly edited) current slice.
3. **User decides to re-slice** — exit. They'll run `/slice`, which reads findings and produces a new plan.

On a happy-path run (no unreviewed findings) this step is invisible. When findings exist, it's the tripwire that catches drift before a Ralph loop burns through slices under a wrong assumption.

Mark the slice `status: in_progress`.

## Read the code

Read the modules the slice names, as they currently exist on the theory branch. Previous slices may have added types, helpers, or conventions the slice artifact couldn't know about. The slice tells you *where* to look; the current code tells you *what* is there.

## Discover tooling

Identify the project's automated feedback loops — test runner, type checker, linter, build. Throughout this skill, **"run the checks"** means all of them. A passing test suite with type errors or lint violations is not green. If a layer isn't configured, propose a minimal setup before starting.

---

## Tracer slice — fire the tracer bullet

**Only if `tracer: true` in the slice frontmatter.** Otherwise skip to the Incremental loop.

Pick the slice's **first behaviour** — the one that cuts end-to-end through every layer the system sketch identifies. Write one test for it.

- Start at the outermost layer the slice's **entry point** names — UI component, HTTP route, CLI entry
- Drive through every layer the slice touches — front-end, request handling, domain logic, persistence
- Real collaborators in-process; mock only at external boundaries (see [mocking.md](mocking.md))
- The production code is lean-but-complete: real structure, real error handling, real checks — just not fully functional yet. Not throwaway. See [tracer-bullets.md](tracer-bullets.md).

Run the full cycle (Red → Green → Refactor → Checklist) on this one test using the rules in the Incremental loop below.

If the tracer fails — the architecture implied by the slice can't carry the behaviour without contortion — stop. Append a finding to `specs/<theory>/findings.md` describing what broke, and exit. Loop back to `/slice` to re-plan; if re-slicing can't find a workable shape, loop back to `/theories`.

Once the tracer passes, continue through the slice's remaining behaviours using the Incremental loop.

---

## Incremental loop

For each remaining behaviour in the slice, run one cycle. Work through **one behaviour at a time**. Do not combine steps.

### Red

Write **ONE** failing test for **ONE** behaviour, using the slice's Given/When/Then or contract assertion verbatim as the spec.

- **Assert first** — write the assertion, then work backwards
- **One outcome per test** — exactly one assertion
- **Usage-driven** — reference the thing in the test before it exists; the compiler/runtime error is the signal to create it
- **Behaviour, not implementation**
- **For `validation: contract` behaviours** — write the structural assertion against the slice's seed fixture, not a specific output value
- **For `validation: human` behaviours** — do not write a test. Note the behaviour in the completion summary for the theory-level validation step.

Run the checks. Confirm the test fails for the *right reason* — a type error or lint violation is not the right reason.

### Green

Write the simplest code that makes the test pass. No more.

Do not introduce abstractions (interfaces, base classes, patterns) until duplication or a failing test forces them.

Mock external dependencies ONLY, according to the rules in [mocking.md](mocking.md); once a double is justified, [test-doubles.md](test-doubles.md) says which kind to reach for.

Run the checks. Type and lint issues in code you just wrote are part of green — fix them now.

### Refactor

Scan the code you just touched against [refactor-candidates.md](refactor-candidates.md) (smell→refactoring lookup) and [shoc.md](shoc.md) (Gorman's SHOC).

Fix issues one at a time. After each refactoring, run the checks. Name the smell fixed and the refactoring applied.

Do not refactor and add behaviour in the same step.

### Quality checklist

Tick each box before declaring the cycle done. Any unchecked box is a defect — fix it before the next Red.

- [ ] **Test describes behaviour** — reads as what the system does, not how
- [ ] **Test uses public interfaces only** — no private methods, no internal state, no back-door access
- [ ] **Test would survive an internal refactor** — rename a private method, extract a helper, swap a data structure: the test still passes
- [ ] **Production code is minimal** — no speculative features, no hypothetical future parameters, no abstractions without a second forcing test
- [ ] **Mocks (if any) cross external boundaries only** — no in-process collaborators doubled
- [ ] **Any mid-cycle discovery about *other* slices** — captured in `findings.md`, tagged with this slice id, not written into any slice file

Report findings to the user: behaviour implemented, test written, refactorings applied, checklist state.

---

## Capturing findings

While working this slice, if you notice something about **other** slices — a missing behaviour, a redundancy, a wrong module boundary, an interface tension — **do not edit those slice files**. Append an entry to `specs/<theory>/findings.md`:

<finding-template>
## <ISO timestamp> — from <this slice id>

**Observation:** <what you noticed>
**Suggests:** <which slice is affected, what might need to change>
**Status:** unreviewed
</finding-template>

Slice files are immutable. Findings are the only channel for cross-slice feedback. The user reviews them at the start of the next `/tdd` cycle.

Literal values inside the *current* slice that turn out to be wrong (e.g. an expected mortgage payment that doesn't match reality) are findings too — do not silently correct them in the slice file. Exit with a finding, let the user decide.

## Completion

When all the slice's behaviours are covered by passing tests (or noted as `validation: human`):

- Mark the slice `status: done` in its frontmatter
- Commit on the theory branch with a message naming the slice
- **Final mock audit** — any remaining doubles should cross external boundaries. If in-process collaborators are mocked, the tracer leaked into horizontal layering — add a finding.
- Summarise: behaviours implemented, design patterns that emerged, `validation: human` behaviours remaining for real-world validation

If this was the **last pending slice** in the theory:

- Propose merging `theory/<theory-slug>` back to main
- Update the spec status to `Done`
- **Validate in the real world** — prompt the user: *"This theory's code is complete — but passing tests only prove the code works as designed, not that the theory is right. Can you validate it with real usage? Does the 'we'll know it worked when' outcome hold? And check any `validation: human` behaviours."*

If the theory didn't hold, loop back to `/theories` to revise.

## Loop-backs

- **`/slice`** — if the tracer fails, if a slice's modules turn out to be wrong mid-cycle, or if findings review decides the slice plan needs revising
- **`/spec`** — if a behaviour can't be written without assumptions the spec doesn't cover, or if the system sketch itself is wrong (e.g. tracer passed but a later slice reveals the architecture can't accommodate it)
- **`/theories`** — if the theory is too large for its slices, clearly doesn't improve the current state as intended, or re-slicing still can't find a workable shape
