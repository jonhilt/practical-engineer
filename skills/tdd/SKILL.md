---
name: tdd
description: Implement one theory through strict outside-in TDD, driven by the vertical slice plan. Use after /slice to drive implementation from the concrete module plan and tracer bullet.
argument-hint: "[optional: path to the theory's Spec]"
---

Write code in small, verified steps. Nothing exists unless a test demands it.

Check the spec's **Requires** field to decide assertion style:

- **Deterministic** — assert specific values
- **Non-deterministic** (LLMs, AI, generative services) — assert **contracts**: structural requirements, content coverage, qualities, and constraints. Properties any correct output would have, regardless of the specific words used.

## Read the spec and slice

Load the spec. It should contain a **Vertical Slice** section from `/slice` naming the concrete modules the slice touches, the entry point the tracer bullet will start from, and the supporting jobs to cover. **Treat this as a design map, not a script.** Only the entry point and layer stack are fixed — they set the direction. The tests themselves emerge from the cycle.

If the spec has unresolved `LLM:` or `API:` dependencies with no Technology Decisions section, stop and run `/spike` first.

If the theory involves multiple layers (especially UI + backend) and no Vertical Slice section exists, stop and run `/slice` first. Do not improvise a starting point that quietly drops the UI layer.

## Discover tooling

Identify the project's automated feedback loops. Check `package.json`, `Makefile`, CI config, or project files for:

- **Test runner** — e.g. `npm test`, `dotnet test`, `pytest`
- **Type checker** — e.g. `tsc --noEmit`, `mypy`, `dotnet build`
- **Linter** — e.g. `eslint`, `ruff`
- **Build step** — if compilation is needed before tests run

Throughout this skill, **"run the checks"** means: run all discovered tools, not just the tests. A passing test suite with type errors or lint violations is not green. If nothing is configured yet, propose a minimal setup — a project without a type checker or linter is missing feedback that prevents avoidable errors.

## How tests emerge

Keep a running list of tests in mind — the slice's headline interaction and supporting jobs give you the first draft. This is Beck's *"running conversation with yourself,"* not a prescribed sequence. Items get added, re-ordered, or deleted as each red-green cycle teaches you what's needed next. Supporting jobs are **requirements scope**, not a 1:1 test map: one behavioural test may cover several jobs, and some jobs only get exercised through outer tests.

Every test verifies **behaviour**, not implementation. Behaviour is what the software does from the outside — inputs, outputs, observable effects through the public interface. Implementation is how it does it — internal structure, method calls, private state, collaborator sequencing.

A mortgage calculator test asserts *"£200,000 at 5% over 25 years produces £1,169.18 monthly"* — stable across any refactoring of the internals. A test that asserts *"calculateCompoundInterest was called before calculateMonthlyPayment"* breaks the moment you refactor, and prevents the refactoring that tests exist to enable. If a test can't survive renaming a private method or extracting a helper, it's coupled to implementation and should be rewritten or deleted.

Start tests at the outer layer the slice identifies (UI or API for user-facing features). Drill inward only when an outer test forces a new unit into existence. Internal helpers don't earn their own tests unless something explicitly demands one.

## The cycle

Work through ONE test at a time. For each test, follow these steps in order. Do not combine steps.

### 1. Red

Propose the test — describe the behaviour it will verify and which job from the spec it exercises. Write a single failing test.

- **Assert first** — write the assertion, then work backwards
- **One outcome per test** — exactly one assertion
- **Usage-driven** — reference the thing in the test before it exists. The compiler/runtime error is your signal to create it.
- **Behaviour, not implementation**

Run the checks. Confirm the test fails for the right reason — a type error or lint violation is not the right reason.

**The first test is a tracer bullet** — a single test that traverses every layer the Vertical Slice lists, beginning at the entry point the slice identifies. If the slice has a UI layer, the tracer bullet starts from the UI (rendering the entry-point component, simulating a user action). Real code however crude, mocks only at external boundaries. TDD chooses what the test actually asserts; the slice just says where it starts.

### 2. Green

Write the simplest code that makes the test pass. No more.

Do not introduce abstractions (interfaces, design patterns, base classes) until duplication or a failing test forces them.

If tempted to mock, first ask whether a three-line real implementation would do. Mock only when the real version drags in another responsibility or crosses an external boundary.

Run the checks. If the type checker or linter flags issues in code you just wrote, fix them now — they're part of green.

### 3. Inspect

Review all code added or changed:

1. **Comprehensible** — Could someone unfamiliar read this?
2. **DRY** — Any duplication? Rule of Three (tolerate two, refactor at three).
3. **Simple** — No nested conditionals, no lengthy methods.

Then apply Gorman's SHOC principles — see [../principles/shoc.md](../principles/shoc.md).

Report findings. If clean, say so and move on.

### 4. Refactor

Fix issues one at a time. After each refactoring, run the checks — tests, types, and lint must all pass. Name the smell fixed and the refactoring applied.

Do not refactor and add behaviour in the same step.

### 5. Confirm

Show the user: the behaviour implemented, the test written, any refactorings applied, current state — all checks passing.

## Completion

When the headline interaction and all supporting jobs are covered by passing tests, summarise what was built, any design patterns that emerged, and remaining inspection concerns.

**Mocked dependencies** — any that remain should only cross external boundaries or be explicitly deferred to a later theory. In-process collaborators should be real. If any aren't, the tracer bullet leaked into horizontal layering — revisit.

**Close out** — update the spec status to `Done`. If using GitHub issues, close the theory issue and tick it in the parent goal's Theories checklist.

**Validate in the real world** — prompt the user: *"This theory's code is complete — but passing tests only prove the code works as designed, not that the theory is right. Can you validate it with real usage? Does the 'we'll know it worked when' outcome hold?"*

If the theory didn't hold, loop back to `/theories` to revise.

## Loop-backs

Go back to `/slice` if a test needs a module the slice plan didn't identify, or implementation reveals a layer that was missed (especially UI).

Go back to `/spec` if a test can't be written without making assumptions the spec doesn't cover, or a job is missing.

Go back to `/theories` if the theory is too large to complete as one thin slice, or clearly doesn't improve the current state as intended.
