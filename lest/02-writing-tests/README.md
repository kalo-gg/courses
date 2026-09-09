# 2. Writing Tests

*Registration happens first, execution second — and almost every rule here follows from that.*

Lest · Easy

---

A describe body runs immediately; an it body does not. Lest collects your whole file, then runs what it collected. This module works through what that two-phase model implies: how a test's full name is built and why duplicates are an error, how the per-test hooks nest and the once-hooks share and tear down, and what the skip, todo and only modifiers actually do.

## Lessons

1. [describe and it](01-describe-and-it.md) — Grouping tests under a label, how a full name is built, and the phase each body runs in.
2. [Per-test hooks](02-each-hooks.md) — beforeEach and afterEach give every test a fresh start, nesting outermost-first and unwinding innermost-first.
3. [Once-hooks and teardown](03-once-hooks.md) — beforeAll and afterAll wrap a block once, run LIFO on the way out, and decide what happens when setup fails.
4. [Skipping and focusing](04-skipping-and-focusing.md) — Parking a test, recording one you have not written, and the modifier that silences a file.
