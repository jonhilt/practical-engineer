# Refactor Candidates

After each green test, scan the code you just touched for these smells. If you find one, apply the paired refactoring before the next Red.

Work one smell at a time. Run the checks after each move. Never refactor and add behaviour in the same step.

## Smell → Refactoring

- **Unclear name** — Rename
- **Magic number / literal** — Replace Magic Number with Symbolic Constant
- **Obscure expression** — Extract Variable (Introduce Explaining Variable)
- **Long method** — Extract Method
- **Comment explaining a block** — Extract Method (let the name replace the comment)
- **Duplication (Rule of Three)** — Extract Method, then Move Method toward the data
- **Feature Envy** (method uses another object's data more than its own) — Move Method
- **Data Clump** (same parameters travelling together) — Introduce Parameter Object
- **Primitive Obsession** — Replace Data Value with Object
- **Long Parameter List** — Introduce Parameter Object / Preserve Whole Object
- **Switch / type-code conditional** — Replace Conditional with Polymorphism
- **Nested conditionals** — Guard Clauses (early return)
- **Null checks scattered across callers** — Introduce Null Object
- **Temp variable used once or computed from fields** — Replace Temp with Query
- **Inheritance used for reuse, not substitution** — Replace Inheritance with Delegation
- **Indirection that no longer earns its keep** — Inline Method / Inline Variable
- **Asking an object for data to make a decision about it** — Tell, Don't Ask (Move Method)
- **Module touching things outside its one job** — Extract Class / Move Method (SHOC: One job)
- **Caller reaching into internals** — Encapsulate Field / Hide Delegate (SHOC: Hides internals)
- **Hard-wired dependency** — Extract Interface / Inject Dependency (SHOC: Swappable)
- **Interface shaped by the implementation** — Reshape around the caller (SHOC: Client-driven)

## Beck's tidyings (smaller than smells)

Cheap structural moves worth making even without a named smell, *before* the next behaviour change:

- Dead code deletion
- Normalize symmetries (make similar code look similar)
- Reading order (top-down in the file)
- Cohesion order (group related things)
- Chunk statements (blank lines between logical groups)
- New interface, old implementation (introduce the API you wish you had, delegate to what exists)

## Stop conditions

- All checks green
- Nothing on this list applies to the code you just touched
- The next move would change behaviour — that belongs in the next Red
