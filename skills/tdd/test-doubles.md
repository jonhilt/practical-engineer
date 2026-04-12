# Test Doubles

Once [mocking.md](mocking.md) has said *yes, a double is justified here*, this doc says *which kind, and how to use it well*.

Meszaros's taxonomy (as popularised by Fowler). Pick the weakest one that does the job — weaker doubles couple the test to less.

## The taxonomy

- **Dummy** — Passed but never used. Fills a parameter slot so the code compiles/runs.
- **Stub** — Returns canned answers to queries. Controls **inputs** to the system under test.
- **Fake** — A real working implementation with a simpler substrate (in-memory repository, in-memory queue). Behaviour is honest; only the backing store is simplified.
- **Spy** — Records what was called on it so the test can assert after the fact. Senses **outputs** from the system under test.
- **Mock** — A spy with expectations set up-front and verified at teardown. Fails the test if the expected calls don't happen.

Pick order (weakest first): **Dummy → Stub → Fake → Spy → Mock**. If a Stub works, don't reach for a Mock.

## Stubs control inputs, Spies sense outputs

Freeman & Pryce's rule from *Growing Object-Oriented Software, Guided by Tests*:

- **Queries** (things the SUT asks for information) → **Stub** them
- **Commands** (things the SUT tells the collaborator to do) → **Spy** on them if the act of calling is the behaviour; otherwise assert on observable state afterwards

Mixing these up — stubbing a command or spying on a query — usually means the collaborator's interface is confused between asking and telling (Command-Query Separation violation).

## Mock roles, not objects

A double stands in for a **role in a conversation**, not for a concrete class. Give the role an interface (even if there's only one real implementation today) and double the interface. This:

- Keeps the test expressing *what the SUT needs*, not *what the implementation offers* (SHOC: Client-driven interfaces)
- Lets the real implementation evolve without rewriting tests
- Makes the role name a design artefact — if you can't name the role, the collaboration isn't clear yet

## Prefer fakes for stateful collaborators

Repositories, caches, session stores, message queues — hand-write a tiny in-memory fake instead of stubbing every method on every test:

- Tests read as domain behaviour, not call-sequence choreography
- One fake amortises across the whole suite
- Contract-test the fake against the real implementation so they can't silently drift

## Avoid interaction-heavy assertions

`verify(mock).foo()` assertions couple tests to implementation. Use them only when *the call itself* is the behaviour under test (sending an email, emitting an event, logging an audit record). For everything else, assert on state observable through the public API.

Red flags:

- Asserting call order unless order is part of the contract
- Asserting argument matchers on internal helper calls
- `verifyNoMoreInteractions` — almost always over-specification
- `never()` assertions — usually a sign you're testing implementation, not behaviour

## Don't mock what you don't own

Never double a third-party class or function directly. Wrap it in your own thin interface (your role name, your method names, your exception types) and double **that**. Reasons:

- Vendor APIs change shape; your wrapper's shape is yours to control
- Your wrapper documents the *role* the third-party plays in your system
- The wrapper becomes the one place that knows the vendor's quirks
- Mocking a vendor API you don't understand fully silently encodes wrong assumptions

## Avoid mock chains

`when(a.getB().getC().doThing()).thenReturn(x)` is a Law of Demeter violation dressed as test setup. Fix the production code — introduce a method on `a` that returns what the caller actually needs — rather than teach the mock framework to traverse the chain.

## Stop conditions

- A weaker double (Dummy < Stub < Fake < Spy < Mock) would work → use it
- You're doubling a concrete class → extract an interface for the role first
- You're doubling a third-party type → wrap it first
- You're asserting on call order or argument matchers for internal helpers → you're testing implementation, redesign or drop the assertion
