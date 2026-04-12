# Practical Engineer

Agent skills for building software through disciplined discovery, specification, and TDD.

## The Skills

Five skills, used in order:

| Skill | Purpose |
|---|---|
| **Goals** | Interview about the problem space, produce a Goals Document |
| **Theories** | Turn the Goals Document into testable theories — hypotheses about what might solve the problem |
| **Spec** | Turn one theory into a clear brief — headline interaction, supporting jobs, and system sketch |
| **Spike** | Resolve technology unknowns from the spec — research options, build throwaway proofs, record decisions |
| **TDD** | Implement one theory through strict outside-in TDD — plan a slice of behaviours, fire a tracer bullet end-to-end, then loop Red/Green/Refactor |

Each skill picks up where the previous one left off. Spike is skipped when all dependencies are deterministic and the team already knows the technology. TDD's own Plan phase keeps the work honest about the full stack — surfacing interface changes, examples, and behaviours to test before any code is written — so a tracer bullet can prove the architecture end-to-end. You don't have to use all five — start wherever your project is.

## Getting started

1. **New project or fuzzy idea** — run `/goals` to interview yourself until the problem is clear, then `/theories` to list what might solve it.
2. **Already have a defined goal** — skip to `/theories`.
3. **Building one theory** — for each theory you want to ship, run `/spec`, then `/spike` (if there are unknown technologies), then `/tdd`.
4. **Loop-backs are expected.** If a later skill reveals its input is wrong, it'll stop and send you back to the skill that needs correcting. Don't fight it.

Skills detect whether you're using GitHub issues or local markdown files and write their artifacts accordingly.

## How the Loops Work

The skills form nested loops. When an inner loop discovers its inputs are wrong, it stops and kicks back to the appropriate outer loop:

```
Goals                         ← outermost: define the problem
  └─ Theories                 ← outer: sequence the theories
       └─ Spec                ← middle: specify one theory
            └─ Spike          ← tech scout: prove technology choices work
                  └─ TDD      ← inner: Plan → Tracer bullet → Loop (Red → Green → Refactor)
```

When a skill discovers its inputs are wrong, it stops and kicks back to the appropriate outer skill — TDD back to Spec when a behaviour can't be written without assumptions the spec doesn't cover, Spec back to Theories when the theory can't be expressed as a single thin slice, and so on. Each skill's own file documents its specific loop-back triggers. Loop-backs are expected, not failures — each pass sharpens the inputs for the next.

## Persistence

Skills write artifacts to local markdown files (`./goals.md`, `./theories.md`, `./specs/*.md`) or to GitHub issues if the project uses them. Each skill picks up where the previous left off by reading the upstream artifact.

Session-level state (progress logs, blockers, parked questions) is intentionally **not** part of these skills. That belongs at the orchestrator layer — a Ralph loop, an agent runtime, or your own note-taking — so the skills stay stateless and composable.

## Principles

Each skill keeps its companion principle files alongside its `SKILL.md`. TDD has the most: `tracer-bullets.md` (Hunt & Thomas), `shoc.md` (Gorman's SHOC), `refactor-candidates.md` (smell→refactoring lookup), `mocking.md`, and `test-doubles.md`.

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
