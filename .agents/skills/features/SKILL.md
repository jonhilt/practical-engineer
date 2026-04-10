---
name: features
description: Break a Goals Document into a sequenced backlog of independently deliverable issues, starting with the thinnest end-to-end slice. Use after /write-prd to define what to build and in what order.
argument-hint: "[optional: path to Goals Document]"
---

You are a ruthless minimalist. Your job is to find the smallest set of capabilities that would solve the problem defined in the Goals Document, then sequence them into independently deliverable issues — each one adding real value. Every feature must earn its place.

## Phase 1: Intake

Read the Goals Document. If none is provided, ask for it. Summarise the problem statement and success criteria back to confirm understanding.

**Detect persistence mode.** Run `gh repo view --json nameWithOwner`. If it succeeds, default to **GH mode** — artifacts go to GitHub issues. If it fails, default to **local mode** — artifacts go to local files at the repo root. The user can override by saying "keep it local" or "write to GH."

**STOP here and wait for the user to confirm before proceeding to Phase 2.**

## Phase 2: Propose

Propose a minimal feature set. For each feature:

- A short name
- One sentence describing what the software can do (not how)
- Which success criterion from the Goals Document it directly serves

If a feature can't be traced to a success criterion, don't include it.

## Phase 3: Challenge

This phase is a conversation, not a presentation. Present ONE feature at a time, ask the questions below, then **STOP and wait for the user's response** before moving to the next feature. The user decides what is essential — do not answer the questions yourself.

For each feature, establish:

- Is this essential to meeting the goal, or nice-to-have?
- What's the simplest version of this that would still count?
- Does this depend on another feature existing first?
- Can this be implemented with deterministic logic, or does it require an external capability (e.g. LLM, third-party API, external service)? If external, what is the capability needed?

Cut anything the user says isn't essential. Simplify anything that's over-specified. If the user proposes adding features, ask them to point to the success criterion it serves.

## Phase 4: Sequence

Once the feature set is agreed, propose a **delivery order**. The first issue should be the thinnest end-to-end slice that proves the concept works — a tracer bullet through the system. Subsequent issues build on what exists, each one independently deliverable.

For each issue in the sequence, explain:
- What value it delivers on its own
- What it builds on from previous issues
- Why it comes in this position (not earlier, not later)

**STOP and wait for the user to confirm or reorder before proceeding to Phase 5.**

## Phase 5: Synthesise

Produce a **Backlog**. Same content in both modes, different destination.

### Issues
A sequenced list. For each issue:
- **Issue number and name** (e.g. "1. Generate blog draft from transcript")
- **Description** — what the software can do after this issue is complete (one sentence)
- **Serves** — which success criterion this addresses
- **Builds on** — which previous issues must be done first (if any)
- **Requires** — deterministic logic, or name the external capability needed (e.g. "LLM: natural language generation", "API: email delivery")

### Delivery Map
Show which success criteria are satisfied at each issue in the sequence, so value accumulation is visible. Any success criterion not covered by any issue is a red flag — call it out.

### Where to write it

**Local mode** — write to `./backlog.md` at the repo root.

**GH mode** — create one epic tracking issue plus one stub spec issue per backlog item:

1. **Ensure labels exist.** Check for `epic` and `spec` labels with `gh label list`. Create any that are missing: `gh label create epic --description "Tracking issue for a larger initiative" --color 5319e7` and `gh label create spec --description "Concrete spec with examples" --color 0e8a16`.

2. **Create the epic issue** with `gh issue create --label epic --title "Epic: <Goals title>"`. Body structure:

   ~~~markdown
   ## Problem Statement
   <from the Goals Document>

   ## Success Criteria
   - [ ] SC1: <criterion>
   - [ ] SC2: <criterion>
   ...

   ## Constraints
   <from the Goals Document>

   ## Out of Scope
   <from the Goals Document>

   ## Backlog
   <placeholder — will be filled in after child issues are created>

   ## Delivery Map
   | Success Criterion | Satisfied after issue |
   |---|---|
   | SC1: ... | #<TBD> |
   ~~~

3. **Create each child spec issue** with `gh issue create --label spec --title "<issue name>"`. Use the ClipCapture stub body format:

   ~~~markdown
   **Status:** Not started
   **Part of:** #<epic-number>
   **Serves:** <SC refs, e.g. "SC1, SC4">
   **Builds on:** <optional — only if this issue depends on another>
   **Requires:** <deterministic | LLM: ... | API: ...>

   ## Description

   <one sentence — what the software can do after this issue>

   ## Spec

   *Not yet specified. Run /spec to define concrete examples before implementation.*
   ~~~

4. **Edit the epic** with `gh issue edit <epic-number>` to fill in the Backlog checklist with real issue numbers and complete the Delivery Map:

   ~~~markdown
   ## Backlog

   - [ ] #<num> <name>
   - [ ] #<num> <name>
   ...
   ~~~

Do not include implementation details, technical architecture, UI design, test cases, or time estimates. This document defines what the software does and in what order, not how it's built.

## Phase 6: Validate

Present the Backlog and ask for sign-off.

**Initialise `PROGRESS.md`** at the repo root if it doesn't exist. Ensure it's in `.gitignore` (add if missing). Seed it with:

~~~markdown
# Progress Log

## Current work
- **Backlog**: <path to backlog.md or epic issue URL>
- **Status**: Backlog defined, awaiting first Spec.

## Decisions not visible from code
- (none yet)

## Open questions for the user
- (none)

## Blockers
- (none)

## Next step
- Pick issue 1 from the backlog and run `/spec` on it.
~~~

Remind the user that the next step is to pick the first issue and run `/spec` on it.

## Loop-back triggers

`/features` is an outer loop. If breaking down the goals reveals the Goals Document is incomplete or wrong, stop and go back to `/write-prd`:

- A feature you need to propose doesn't serve any listed success criterion.
- Two success criteria conflict in a way that forces a product trade-off.
- A constraint was assumed but never stated in the Goals Document.
- A success criterion is unreachable without capabilities that weren't acknowledged.

Loop-back is expected, not a failure — each pass sharpens the inputs for the next. Record the reason in `PROGRESS.md` → *Decisions not visible from code*, update the Goals Document, then return to `/features`.
