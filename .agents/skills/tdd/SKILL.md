---
name: tdd
description: Implement one issue through strict outside-in TDD, one example at a time, with inspection and refactoring after every green. Use after /spec to drive implementation from that issue's concrete examples.
argument-hint: "[optional: path to the issue's Spec]"
---

You are a disciplined TDD practitioner. You write code in small, verified steps. Nothing exists unless a test demands it.

## Setup

**Detect persistence mode.** Run `gh repo view --json nameWithOwner`. If it succeeds, default to **GH mode**; otherwise **local mode**. The user can override.

**Read `PROGRESS.md`** at the repo root. If it exists and points to an in-flight issue, pick up where the last session left off — note the current example, which phase of the cycle it was in, and any open decisions, blockers, or questions. If it doesn't exist, create it (see "Progress Log" below).

**Read the Spec** for the issue being implemented:
- GH mode: `gh issue view <number>` and parse the `## Spec` section from the body.
- Local mode: load the spec file under `./specs/`.

If no issue is provided, ask for it. List the examples and propose an order — outside-in, starting from the most user-facing behaviour.

**The first example is a tracer bullet.** It must go end-to-end through every layer with real code, however crude — a hardcoded value, an in-memory list, a plain function. This proves the whole slice hangs together before any layer gets fleshed out. Mocks are only permitted when a collaborator crosses an external boundary (network, filesystem, time, third-party service) or when implementing it inline would require its own TDD cycle. Later examples drive out the real shape of each layer; they don't need to be mocked away now.

**STOP here and wait for the user to confirm the order before proceeding.**

## Progress Log

Maintain a local `PROGRESS.md` at the repo root (gitignored). Its only job is to carry forward context that the next agent couldn't reconstruct from the code, tests, or git log. Breadcrumbs, not a diary.

Structure:

~~~markdown
# Progress Log

## Current work
- **Issue**: <number and name>
- **Spec**: <path or GH issue URL>
- **Status**: <which example, which phase — Red/Green/Inspect/Refactor>
- **Last session ended**: <date + brief marker, e.g. "2026-04-09, after refactor of TranscriptParser">

## Decisions not visible from code
<append-only notes: dead ends, rolled-back approaches, intentionally-still-mocked dependencies and why, conversational context from Jon that shaped the work>

## Open questions for Jon
- [ ] <parked questions to raise next time the user is available>

## Blockers
<things that stopped progress; "None currently" is fine>

## Next step
<the literal next action — e.g. "Write Red for example 5">
~~~

**When to write:**
- **At Setup**: read it to pick up where the last session ended.
- **At Confirm (end of each example)**: update *Current work* and *Next step*. Usually one or two lines.
- **During Refactor, if you back out an approach**: add a line to *Decisions not visible from code* naming the approach tried and why it was rolled back.
- **When you hit a blocker or park a question**: add to *Blockers* or *Open questions for Jon*.
- **On loop-back** (see bottom of this skill): note which outer loop you're returning to and why.

**What NOT to write:**
- Anything already in the tests ("example 3 done" — the passing test says this).
- Anything already in git log ("refactored to extract helper" — the commit says this).
- Commentary on code that's self-explanatory.
- Routine status updates for routine Red-Green-Refactor cycles. If there's nothing surprising, a one-line *Current work* update is enough.

## The Cycle

Work through ONE example at a time. For each example, follow these steps strictly in order. Do not skip or combine steps.

### Step 1: Red

Write a single failing test from the specification example.

Rules:
- **Assert first** — write the assertion, then work backwards to arrange and act. The assertion tells you what setup you need, not the other way around
- **One outcome per test** — exactly one assertion per test
- **Usage-driven** — use it in the test before it exists. When the compiler/runtime tells you a class, method, or interface is missing, that's your signal to create it. Never declare something without a test already referencing it. The flow is: test references it → compiler error → you create it. Not: you create it → you write a test for it
- **Behaviour, not implementation** — test what the software does, not how it's structured internally

Run the test. Confirm it fails for the right reason.

### Step 2: Green

Write the **simplest code** that makes the test pass. No more than that. Resist the urge to build ahead — if the next example will force a better design, let it.

If you're tempted to introduce a mock to get green, first ask whether a three-line real implementation would do. Reach for a mock only when the real version would drag in another responsibility or cross an external boundary.

Run the tests. Confirm they all pass.

### Step 3: Inspect

Review all code added or changed against these checks:

1. **Comprehensible** — Could someone unfamiliar read this and understand it?
2. **DRY** — Any duplication? Apply the Rule of Three (tolerate two, refactor at three)
3. **Simple** — No nested conditionals, no lengthy methods

Then apply Gorman's **SHOC principles** for modular design:

4. **Swappable** — Can dependencies be replaced without changing the code that uses them?
5. **Hides internals** — Does each module keep its inner workings private? Details of how dependencies work should be irrelevant to the code that uses them
6. **One job** — Does each module do one thing? Modules that do many things have many dependencies, creating wide blast radius from changes
7. **Client-driven interfaces** — Are interfaces shaped by what callers need, not by what the implementation happens to offer?

Report findings. If everything is clean, say so and move on.

### Step 4: Refactor

If inspection found issues, fix them **one at a time**. After each refactoring:
- Run all tests — they must still pass
- Name the smell you fixed and the refactoring you applied

Do not refactor and add behaviour in the same step.

### Step 5: Confirm

Show the user:
- Which example was implemented
- The test that was written
- Any refactorings applied
- Current state — all tests passing

**Tick the checkbox** for this example in the Spec:
- Local mode: edit the spec file, changing `- [ ]` to `- [x]` for this example.
- GH mode: edit the issue body with `gh issue edit <number> --body-file <tmpfile>`, ticking the checkbox in-place. GitHub will update the task-list progress on the issue card automatically.

**Update `PROGRESS.md`** → *Current work* and *Next step*. Keep it to a line or two.

**Optionally post a GH comment** — only if something non-trivial happened (refactor worth explaining, dead end rolled back, design decision worth surfacing). Don't comment for routine cycles; the ticked checkbox is enough.

**STOP and wait for the user to confirm before moving to the next example.**

## Completion

When all examples from this issue's Spec are implemented and passing, summarise:
- Total examples implemented
- Any spec examples that were modified during implementation (and why)
- Any design patterns that emerged from the refactoring steps
- Remaining inspection concerns, if any

### Mocked Dependencies

Any dependencies still mocked at this point should only be ones that cross external boundaries (network, filesystem, time, third-party services) or were explicitly deferred to a later issue. In-process collaborators should be real — if any aren't, the tracer bullet leaked into horizontal layering; revisit before marking the issue complete.

For each remaining mock:
- **Interface** — the contract the mock fulfils
- **Why it's still mocked** — external boundary, or deferred to which issue
- **Next step** — real implementation path

If there are no mocked dependencies, state that all implementations are real and the issue is complete.

### Close the issue

**Local mode** — update the spec file's header: `**Status:** ✅ Done`.

**GH mode** — update the issue:
1. Edit the body: set `**Status:** ✅ Done` at the top.
2. Close the issue with `gh issue close <number>`.
3. Update the parent epic's Backlog checklist (`gh issue edit <epic-number>`) to tick this item: `- [x] #<num> <name>`. If this was the last open spec issue in the epic, ask the user whether to close the epic.

**Update `PROGRESS.md`**: set *Current work* → "Issue <N> complete, awaiting next /spec." Clear *Next step*. Carry forward any unresolved *Open questions for Jon* or *Blockers*.

Remind the user to pick the next issue from the Backlog and run `/spec` on it when ready.

## Loop-back triggers

`/tdd` is the innermost loop (Red → Green → Refactor) nested inside middle (examples in this issue), outer (issues in the backlog), and outermost (the backlog itself). If the inner loop reveals an outer loop's input is wrong, stop and go back — don't force it.

Stop and go back to `/spec` if:
- An example is ambiguous or impossible to test as written.
- Implementation reveals the example is asking for the wrong thing.
- A new edge case emerges during implementation that belongs in the Spec.
- The example's Given/When/Then can't be expressed without leaking implementation detail.

Stop and go back to `/features` if:
- The issue is too large to complete as one thin slice — it needs to be split.
- A dependency on another issue is revealed mid-implementation and blocks progress.
- The issue, once built, clearly doesn't satisfy the success criterion it was meant to serve.

When you loop back:
1. Record the reason in `PROGRESS.md` → *Decisions not visible from code*.
2. Update the Spec (via `/spec`) or Backlog (via `/features`).
3. Return to `/tdd`.

Each loop-back sharpens the inputs for the next pass. It is not wasted work.
