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

Load the spec. It should contain a **Vertical Slice** section from `/slice` naming the concrete modules the slice touches, the tracer bullet in codebase terms, and the TDD order. **Use this as the literal plan for TDD.**

If the spec has unresolved `LLM:` or `API:` dependencies with no Technology Decisions section, stop and run `/spike` first.

If the theory involves multiple layers (especially UI + backend) and no Vertical Slice section exists, stop and run `/slice` first. Do not improvise a testing order that quietly drops the UI layer.

## Discover tooling

Identify the project's automated feedback loops. Check `package.json`, `Makefile`, CI config, or project files for:

- **Test runner** — e.g. `npm test`, `dotnet test`, `pytest`
- **Type checker** — e.g. `tsc --noEmit`, `mypy`, `dotnet build`
- **Linter** — e.g. `eslint`, `ruff`
- **Build step** — if compilation is needed before tests run

Throughout this skill, **"run the checks"** means: run all discovered tools, not just the tests. A passing test suite with type errors or lint violations is not green. If nothing is configured yet, propose a minimal setup — a project without a type checker or linter is missing feedback that prevents avoidable errors.

## The cycle

Work through ONE test at a time. For each test, follow these steps in order. Do not combine steps.

### 1. Red

Propose the test — describe the behaviour it will verify and which job from the spec it exercises. Write a single failing test.

- **Assert first** — write the assertion, then work backwards
- **One outcome per test** — exactly one assertion
- **Usage-driven** — reference the thing in the test before it exists. The compiler/runtime error is your signal to create it.
- **Behaviour, not implementation**

Run the checks. Confirm the test fails for the right reason — a type error or lint violation is not the right reason.

**The first test is the tracer bullet named in the Vertical Slice.** It must traverse every layer the slice lists — if the slice has a UI layer, it starts from the UI. Real code however crude, mocks only at external boundaries.

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
