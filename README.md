# Practical Engineer

Agent skills for building software through disciplined discovery, specification, and TDD.

## The Skills

Six skills, used in order:

| Skill | Purpose |
|---|---|
| **Goals** | Interview about the problem space, produce a Goals Document |
| **Theories** | Turn the Goals Document into testable theories — hypotheses about what might solve the problem |
| **Spec** | Turn one theory into a clear brief — headline interaction, supporting jobs, and napkin sketch |
| **Spike** | Resolve technology unknowns from the spec — research options, build throwaway proofs, record decisions |
| **Slice** | Map the spec onto concrete modules in the codebase — identify the full vertical slice (UI, API, domain, data) and name the tracer bullet |
| **TDD** | Implement one theory through strict outside-in TDD, driven by the slice plan |

Each skill picks up where the previous one left off. Spike is skipped when all dependencies are deterministic and the team already knows the technology. Slice keeps TDD honest about the full stack — preventing it from quietly dropping the UI layer and drifting to backend-only tests. You don't have to use all six — start wherever your project is.

## How the Loops Work

The skills form nested loops. When an inner loop discovers its inputs are wrong, it stops and kicks back to the appropriate outer loop:

```
Goals                         ← outermost: define the problem
  └─ Theories                 ← outer: sequence the theories
       └─ Spec                ← middle: specify one theory
            └─ Spike          ← tech scout: prove technology choices work
                 └─ Slice     ← map spec onto codebase: modules + tracer bullet
                      └─ TDD  ← inner: Red → Green → Inspect → Refactor → Confirm
```

**TDD → Slice** — a test needs a module the slice plan didn't identify, or implementation reveals a missed layer (especially UI).

**TDD → Spike** — unresolved technology dependencies in the spec that need proving before tests can be written.

**TDD → Spec** — a test can't be written without assumptions the spec doesn't cover, or a job is missing from the spec.

**TDD → Theories** — the theory is too big for one slice, has an unmet dependency, or doesn't improve the current state as expected.

**Slice → Spec** — supporting jobs are too vague to map onto modules, or the napkin sketch assumes a data flow the codebase can't support.

**Slice → Spike** — planned modules depend on an unvalidated or incorrect technology choice.

**Slice → Theories** — the slice is too large to implement as one thin feature, or its cost is disproportionate to the theory's value.

**Spike → Spec** — spiking reveals the headline interaction isn't feasible with available technology, or a supporting job needs redesigning.

**Spike → Theories** — the technology required is prohibitively complex or expensive, making the theory not worth pursuing.

**Spec → Theories** — the theory can't be described as a single thin slice, or the napkin sketch reveals it requires capabilities from a different theory.

**Theories → Goals** — a theory doesn't address the problem statement, or the current state baseline is incomplete.

Loop-backs are expected, not failures. Each pass sharpens the inputs for the next.

## Persistence

Skills write artifacts to local markdown files (`./goals.md`, `./theories.md`, `./specs/*.md`) or to GitHub issues if the project uses them. Each skill picks up where the previous left off by reading the upstream artifact.

Session-level state (progress logs, blockers, parked questions) is intentionally **not** part of these skills. That belongs at the orchestrator layer — a Ralph loop, an agent runtime, or your own note-taking — so the skills stay stateless and composable.

## Shared principles

Some concepts are referenced by multiple skills and live in `skills/principles/`:

- [`shoc.md`](skills/principles/shoc.md) — Gorman's SHOC principles (Swappable, Hides internals, One job, Client-driven interfaces), used during TDD inspection and refactoring.

## Install

### Via [skills.sh](https://skills.sh)

```sh
npx skills add jonhilt/practical-engineer
```

### Manual

Each skill is a standalone markdown file under `skills/`. Copy them into your project or point your agent tooling at them — they have no dependencies beyond a `gh` CLI for GitHub mode.

```sh
cp -r practical-engineer/skills /path/to/your/project/.agents/
```

## License

MIT
