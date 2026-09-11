# 5. Iterators & Closures

*Functions that remember, loops you write yourself, and the list transformations built from both.*

Luau Intermediate · Medium

---

A closure is a function that remembers where it was made, and that one idea produces counters, iterators and most of the small helpers a codebase accumulates. This module writes iterators by hand, hooks them into the generic for with a metamethod, and then builds the list transformations that every language eventually grows.

## Lessons

1. [Functions that remember](01-closures-as-state.md) — A closure captures the variable, which makes it the smallest object Luau has.
2. [Writing an iterator](02-custom-iterators.md) — The generic for calls a function until it returns nil, which is the whole protocol you must satisfy.
3. [Looping an object directly](03-iter-metamethod.md) — Give a table an iter metamethod and the plain generic for works on it, with no helper at the call site.
4. [map, filter and reduce, by hand](04-transforming-lists.md) — Luau ships no map or filter, so every codebase writes them; here is the version worth copying.
