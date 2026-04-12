---
name: tdd
description: Implement one theory through strict outside-in TDD in three phases — plan a slice of behaviours, fire a tracer bullet end-to-end, then loop Red/Green/Refactor through the remaining behaviours. Use after /spec (or /spike if the spec has technology unknowns).
argument-hint: "[optional: path to the theory's Spec]"
---

Implement one theory through strict outside-in TDD in three phases: **Plan → Tracer bullet → Loop**.

Every test verifies **behaviour**, not implementation. Behaviour is what the software does from the outside — inputs, outputs, observable effects through the public interface. Implementation is how it does it — internal structure, private state, which collaborator was called.

A mortgage calculator test asserts *"£200,000 at 5% over 25 years produces £1,169.18 monthly"* — stable across any refactoring of the internals. **Do not write tests that are coupled to implementation.**

## Read what you need

Load the spec. It names the **headline interaction**, the **supporting jobs**, and a **system sketch** of the responsibilities and collaborations. You'll turn these into a concrete slice during Phase 1.

Check the spec's **Requires** field to decide assertion style:

- **Deterministic** — assert specific values
- **Non-deterministic** (LLMs, AI, generative services) — assert **contracts**: structure, coverage, qualities, constraints any correct output would have regardless of specific words used

## Discover tooling

Identify the project's automated feedback loops — test runner, type checker, linter, build. Throughout this skill, **"run the checks"** means all of them. A passing test suite with type errors or lint violations is not green. If a layer isn't configured, propose a minimal setup before starting.

---

## Phase 1 — Plan (stop for approval)

Do not write any test until the user has approved this plan. Write it as a short message back to the user with four sections.

### 1. Interface changes

Name the thin interface this theory will introduce or change. Aim for deep modules, with thin interfaces which hide the implementation details,

Depending on the app this interface may be an API route, vertical slice handler, UI entry points. No internal types, no private helpers. What does the *outside* of the change look like?

Don't guess, get the user to confirm this is the shape the user expects. A wrong interface caught here costs nothing; caught after the tracer bullet, it costs a rewrite.

### 2. Behaviours to test

List the observable behaviours this theory must exhibit, phrased from the outside:

- Good: *"When a user submits the form with a valid email, they see a confirmation screen."*
- Bad: *"UserService.save is called with the form data."*

The headline interaction from the spec is the first entry. Supporting jobs fan out from there. This list is a draft — Beck's *"running conversation with yourself"* — items will be added, reordered, or deleted as cycles teach you what's needed.

### Present and wait

Show the user the plan. Wait for explicit approval. Revise if asked. Do not proceed to Phase 2 without it.

---

## Phase 2 — Tracer bullet

Write **one test that confirms one assumption**: that the architecture implied by the Plan actually carries the headline behaviour end-to-end.

- Start at the outermost layer the Plan's **interface changes** name — UI component, HTTP route, or CLI entry point
- Drive through every layer the spec's **system sketch** identifies — front-end, request handling, domain logic, persistence if relevant
- Real collaborators in-process; mock only at external boundaries
- The production code is lean-but-complete: real structure, real error handling, real checks — just not fully functional yet. Not throwaway. See [tracer-bullets.md](tracer-bullets.md).

Run the full TDD cycle from Phase 3 (Red → Green → Refactor → Checklist) on this one test.

If the assumption doesn't hold — the architecture can't carry the behaviour without contortion — stop and **re-plan** (return to Phase 1). If re-planning can't find a workable shape either, loop back to `/theories` — the theory itself may not fit. Don't muscle through.

---

## Phase 3 — Incremental loop

For each remaining behaviour on the Plan, run one cycle. Work through **one behaviour at a time**. Do not combine steps.

### Red

Write **ONE** failing test for **ONE** behaviour.

- **Assert first** — write the assertion, then work backwards
- **One outcome per test** — exactly one assertion
- **Usage-driven** — reference the thing in the test before it exists; the compiler/runtime error is the signal to create it
- **Behaviour, not implementation**

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

Report findings to the user: behaviour implemented, test written, refactorings applied, checklist state.

---

## Completion

When the headline interaction and all planned behaviours are covered by passing tests, summarise what was built, any design patterns that emerged, and remaining concerns.

**Final mock audit** — any remaining doubles should cross external boundaries or be explicitly deferred to a later theory. If in-process collaborators are mocked, the tracer bullet leaked into horizontal layering — revisit.

**Close out** — update the spec status to `Done`. If using GitHub issues, close the theory issue and tick it in the parent goal's Theories checklist.

**Validate in the real world** — prompt the user: *"This theory's code is complete — but passing tests only prove the code works as designed, not that the theory is right. Can you validate it with real usage? Does the 'we'll know it worked when' outcome hold?"*

If the theory didn't hold, loop back to `/theories` to revise.

## Loop-backs

- **`/spec`** — if a test can't be written without assumptions the spec doesn't cover, a behaviour/example is missing, or the system sketch is too vague to identify layers
- **`/theories`** — if the theory is too large for one thin slice, clearly doesn't improve the current state as intended, or the tracer bullet shows it can't be carried end-to-end even after re-planning
