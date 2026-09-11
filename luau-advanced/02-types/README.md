# 2. The Type System, Deeply

*Refinements, intersections, variance and the type functions that compute types.*

Luau Advanced · Medium

---

Luau Intermediate covered the annotations you write every day. This module covers the ones you write when the easy answer does not fit: refinements that narrow a value as the code proves things about it, intersections and the variance rules that decide what is assignable to what, and type functions, which compute a type from another type at check time.

## Lessons

1. [Narrowing a value](01-refinements.md) — Guards teach the checker what a value is, and a call between the guard and the use can unteach it.
2. [Intersections](02-intersections.md) — A & B is a value that is both shapes at once, and an intersection of arrows is how Luau spells an overload.
3. [What is assignable to what](03-variance.md) — Mutable containers are invariant, read-only ones are not, and function parameters go the other way.
4. [Types that compute types](04-type-functions.md) — type function runs at check time, and keyof and index derive a type from another one.
