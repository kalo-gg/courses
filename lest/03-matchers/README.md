# 3. Matchers

*Jest's vocabulary, Luau's semantics — and the handful of places those differ.*

Lest · Medium

---

expect(value) returns an object carrying every matcher. Most do what their name says; the ones worth learning precisely are where Luau is not JavaScript: what counts as truthy, what toEqual does with NaN, why toMatch takes a pattern rather than a substring, and why toThrow takes a function. The module closes with the first project: a spec suite for the shop's cart.

## Lessons

1. [Equality and truthiness](01-equality.md) — Two kinds of equal, and two values that are truthy here and falsy elsewhere.
2. [Strings, collections and types](02-strings-and-collections.md) — toContain, toHaveLength, toMatch and toBeA, and the one that takes a pattern rather than a substring.
3. [Numbers and chaining](03-numbers-and-chaining.md) — Ordering matchers, toBeCloseTo and its precision, and reading several claims about one value as one chain.
4. [Errors and matcher misuse](04-errors-and-misuse.md) — Why toThrow takes a function, matching on the message, and what a misuse says about your test.
