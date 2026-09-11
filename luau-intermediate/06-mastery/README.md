# 6. Structuring a Codebase

*Module boundaries, errors that callers can handle, tests, and an API someone else can use.*

Luau Intermediate · Hard

---

Everything so far fits in one file. A codebase does not, and the decisions that keep it readable are about boundaries: what a module exposes, how it reports a failure to a caller who has to handle it, how it is tested, and what its API asks of the person using it. This module makes those decisions explicit, then builds the whole inventory system.

## Lessons

1. [What a module exposes](01-module-boundaries.md) — Everything not in the returned table is private, and that is the only privacy Luau has.
2. [Failures a caller can handle](02-errors-as-values.md) — error is for a bug, a returned ok and message is for a failure the caller was always going to hit.
3. [Testing what you wrote](03-testing-a-module.md) — A test is a function that asserts; a framework only organises them, and assert alone gets you started.
4. [Designing the thing someone else calls](04-api-design.md) — Fewer arguments, obvious names, no surprising mutation, and a return that says what happened.
