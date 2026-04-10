# Practical Engineer

Agent skills for building software through disciplined discovery, specification, and TDD.

## The Skills

Four skills, used in order:

| Skill | Purpose |
|---|---|
| **Write PRD** | Interview about the problem space, produce a Goals Document |
| **Features** | Break the Goals Document into a sequenced backlog of thin, deliverable issues |
| **Spec** | Define one issue's behaviour through concrete Given/When/Then examples |
| **TDD** | Implement one issue through strict outside-in TDD, one example at a time |

Each skill picks up where the previous one left off. You don't have to use all four — start wherever your project is.

## How the Loops Work

The skills form nested loops. When an inner loop discovers its inputs are wrong, it stops and kicks back to the appropriate outer loop:

```
Write PRD           ← outermost: define the problem
  └─ Features       ← outer: sequence the backlog
       └─ Spec      ← middle: specify one issue
            └─ TDD  ← inner: Red → Green → Inspect → Refactor → Confirm
```

**TDD → Spec** — an example is ambiguous, wrong, or a new edge case emerges during implementation.

**TDD → Features** — the issue is too big for one slice, has an unmet dependency, or doesn't actually serve its success criterion.

**Spec → Features** — the issue can't be described as a single thin slice, or its examples reveal a sequencing problem.

**Features → Write PRD** — a feature can't be traced to a success criterion, or constraints conflict.

Loop-backs are expected, not failures. Each pass sharpens the inputs for the next.

## Persistence

Skills auto-detect whether you have a GitHub repo (`gh repo view`) and default to **GH mode** (issues, labels, task-list checkboxes) or **local mode** (markdown files in the repo). You can override either way.

A gitignored `PROGRESS.md` file carries forward context between sessions — current work, parked questions, blockers, and decisions that aren't visible from code or git history.

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
