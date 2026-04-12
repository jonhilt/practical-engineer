# Mocking Rules

When a test needs a collaborator, the default is **use the real thing**. Reach for a test double only when one of the rules below demands it. Every mock you introduce is coupling to implementation you'll have to maintain.

For *which kind* of double to reach for once you've decided you need one, see [test-doubles.md](test-doubles.md).

## Only mock at external boundaries

Mock when the real dependency crosses one of these lines:

- **Network** — HTTP, gRPC, message brokers, third-party APIs
- **Filesystem** — disk I/O
- **Database** — unless an in-memory variant exists and is fast
- **Time** — `now()`, timers, scheduling
- **Randomness** — `Math.random`, UUIDs, crypto randomness
- **Process / OS** — shell commands, environment
- **LLMs and other non-deterministic services**

In-process collaborators (pure functions, value objects, domain logic, other classes in your module) are **not** boundaries. Use the real thing.

## Don't mock

- **What you don't own.** Wrap the third-party library in your own interface first, then mock the wrapper. This keeps the mock contract under your control and gives you a seam that survives vendor churn.
- **Value objects, DTOs, records.** They're cheap to construct.
- **The system under test.** If you're reaching for a partial mock of the thing you're testing, the class has more than one job — Extract Class instead.
- **Pure functions.** Call them directly.
- **Things only to avoid writing a three-line real implementation.** Build the real thing.

## Listen to the tests

Painful mocking is a **design signal**, not a testing problem. If you notice any of these, stop and redesign — don't make the mocks work:

- You need more than ~2 doubles to test one behaviour → the unit has too many collaborators (Feature Envy / One job violation)
- A mock setup is longer than the test body → coupling to implementation
- You're mocking chains like `mock.a().b().c()` → Law of Demeter violation, Tell-Don't-Ask
- The test breaks whenever you rename an internal method → you're asserting interactions you shouldn't care about
- You can't write the test without specifying call order → the collaborator protocol is too rich
- You had to make a private method public or add a getter *for the test* → SHOC: Hides internals violated

When this happens, the fix is in the production code (Extract Class, Move Method, Introduce Parameter Object, Tell-Don't-Ask), not the test.

## State over interaction

Prefer assertions about *observable outcomes* over assertions about *which methods were called*. 

Use interaction verification only when the *act of calling* is the behaviour — sending an email, publishing an event, writing to a log.