# Practical Engineer

Agent skills for building software through disciplined discovery, specification, and TDD — designed for use with [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## The Skills

Four skills, used in order:

| Skill | Command | Purpose |
|---|---|---|
| **Write PRD** | `/write-prd` | Interview about the problem space, produce a Goals Document |
| **Features** | `/features` | Break the Goals Document into a sequenced backlog of thin, deliverable issues |
| **Spec** | `/spec` | Define one issue's behaviour through concrete Given/When/Then examples |
| **TDD** | `/tdd` | Implement one issue through strict outside-in TDD, one example at a time |

Each skill picks up where the previous one left off. You don't have to use all four — start wherever your project is.

## How the Loops Work

The skills form nested loops. When an inner loop discovers its inputs are wrong, it stops and kicks back to the appropriate outer loop:

```
/write-prd          ← outermost: define the problem
  └─ /features      ← outer: sequence the backlog
       └─ /spec     ← middle: specify one issue
            └─ /tdd ← inner: Red → Green → Inspect → Refactor → Confirm
```

**`/tdd` → `/spec`** — an example is ambiguous, wrong, or a new edge case emerges during implementation.

**`/tdd` → `/features`** — the issue is too big for one slice, has an unmet dependency, or doesn't actually serve its success criterion.

**`/spec` → `/features`** — the issue can't be described as a single thin slice, or its examples reveal a sequencing problem.

**`/features` → `/write-prd`** — a feature can't be traced to a success criterion, or constraints conflict.

Loop-backs are expected, not failures. Each pass sharpens the inputs for the next.

## Persistence

Skills auto-detect whether you have a GitHub repo (`gh repo view`) and default to **GH mode** (issues, labels, task-list checkboxes) or **local mode** (markdown files in the repo). You can override either way.

A gitignored `PROGRESS.md` file carries forward context between sessions — current work, parked questions, blockers, and decisions that aren't visible from code or git history.

## Setup

Copy the `.agents/` directory into your project, or clone this repo and symlink it:

```sh
# Option 1: copy into your project
cp -r practical-engineer/.agents /path/to/your/project/

# Option 2: clone and reference
git clone https://github.com/jonhilt/practical-engineer.git
```

The skills will be available as `/write-prd`, `/features`, `/spec`, and `/tdd` in Claude Code.

## License

MIT
