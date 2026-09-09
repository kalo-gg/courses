# 6. Suite Design & Capstone

*Tests that say what they mean, doubles you wrote yourself, and a suite that stays green for the right reasons.*

Lest · Hard

---

Everything so far was mechanism; this module is judgment. What a test should claim and how it should be named, how to isolate code from the clock and the network with fakes you write by hand — Lest ships no mocking API on purpose — why suites go flaky and how to quarantine one honestly, and how to plan a suite for a real project. It ends with the capstone: the checkout suite, gated in CI.

## Lessons

1. [What a test should say](01-test-design.md) — One behavior per test, a name that reads as a sentence, and asserting on the result rather than the route.
2. [Fakes you write yourself](02-test-doubles.md) — Lest ships no mocking API: inject the clock, the stock and the mailer, and hand a spec a fake it can inspect.
3. [Flaky tests](03-flaky-tests.md) — Where nondeterminism creeps in, and how to quarantine a test honestly when you cannot fix it today.
4. [Planning a suite](04-planning-a-suite.md) — Which suite each module belongs in, a coverage number you can hold, and a plan you can run as todos.
