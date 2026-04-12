# Tracer Bullets

From *The Pragmatic Programmer* by Hunt & Thomas — Tip #20: *Use Tracer Bullets to Find the Target*.

## The metaphor

Gunners don't always have the firing table. When the target is moving or the parameters are unknown, they load tracer rounds — phosphorus-tipped bullets that glow in flight so you can *see* where they're landing and adjust aim in real time. Quick feedback means less correction to stay on target.

Software has the same problem: you rarely know all the parameters up front (requirements, user reactions, environment). A tracer bullet is the software equivalent of a round you can see.

## What a tracer bullet IS

Production code that's **lean but complete** — a thin slice that goes end-to-end through every layer of the system (UI → business logic → data, whatever the stack is). Each layer's implementation can be crude, but it's real code with real structure, real error handling, real checks.

> *"Tracer code is not disposable: you write it for keeps. It contains all the error checking, structuring, documentation, and self-checking that any piece of production code has. It simply is not fully functional."*

## What a tracer bullet is NOT

- **Not a prototype.** Prototypes are disposable learning exercises. Tracer bullets are production code that forms the skeleton of the final system.
- **Not hardcoded throwaway.** Stubs get replaced as subsequent work fills in real behaviour — but the skeleton stays.
- **Not a test.** The test drives the tracer bullet into existence. The tracer bullet is the *code*.

## What it's FOR

Five advantages Hunt & Thomas list:

1. **Users see something working early**
2. **Developers build a structure to work in**
3. **You have an integration platform** — continuous, not big-bang
4. **You have something to demonstrate**
5. **You have a better feel for progress**

The underlying purpose is **feedback**: a runnable end-to-end slice lets you see whether you're aimed at the right target and adjust before firing more rounds.

## How it grows

Subsequent work *extends* the tracer bullet rather than replacing it — completing stubbed routines, adding new behaviour, refining each layer. The skeleton stays; the flesh grows.

> *"Add to this framework with new functionality, completing stubbed routines."*
