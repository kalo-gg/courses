# 2. Structs and enums

*Named shapes with defaults and methods, enums that carry payloads, a `match` that must cover every case, and bindings that test and name a value in one step.*

Alloy · Easy

---

Luau gives you tables and leaves the shape to you. This module gives shapes names. A `struct` declares fields and defaults, `new` checks every construction at compile time, and `impl` adds methods. An `enum` lists a type's variants: unit variants are plain strings, and payload variants carry data. `match` takes a value apart and refuses to compile when a variant has no arm. Conditional bindings (`if const`, `while const`, the guard clause and the refutable `const`) test and name a value in one step. The module ends with a loop that drains a queue of commands one variant at a time.

## Lessons

1. [Structs and impl](01-structs.md) — Declare a named shape with defaults, construct it with `new` so the compiler checks every field, and give it methods with `impl`.
2. [Enums](02-enums.md) — List a type's cases with `enum`: unit variants that are plain strings, payload variants that carry data, and methods called on the enum.
3. [Match](03-match.md) — Take a value apart with `match`, as a statement or an expression, and let the compiler reject a match that misses a variant.
4. [Conditional bindings](04-bindings.md) — Bind a value and test it in one step with `if const`, `while const`, the guard clause and the refutable `const`, and know what each does when the test fails.
