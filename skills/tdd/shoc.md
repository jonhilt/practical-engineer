# SHOC Principles

Jason Gorman's principles for well-designed modules. Apply during code inspection and refactoring.

- **Swappable** — Dependencies can be replaced without changing the code that uses them.
- **Hides internals** — Each module keeps its inner workings private. Callers can't reach in.
- **One job** — Each module does one thing. One reason to change.
- **Client-driven interfaces** — Interfaces are shaped by what callers need, not by what the implementation offers.

Use these alongside the fundamentals: comprehensibility, DRY (Rule of Three — tolerate two, refactor at three), and simplicity (no nested conditionals, no sprawling methods).

Abstractions, interfaces, and design patterns emerge from refactoring when duplication or a failing test forces them — not up front.
