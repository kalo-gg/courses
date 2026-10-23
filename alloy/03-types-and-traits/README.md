# 3. Types, traits and extensions

*Shapes with `interface`, behaviour with traits, generated methods with derives, and methods on types you do not own.*

Alloy · Medium

---

Structs give a shape a class. This module covers the rest of Alloy's type tools. `interface` and the utility types name a shape at no run-time cost. Traits state the methods a type must have, and the operator traits turn methods into metamethods. `@derive` writes equality, ordering, cloning and saving from the field list, and `@sealed` turns a typo'd write into an error. The module ends with `export impl`, which adds methods to types you do not own, such as `Vector3` and `string`, and with how the compiler dispatches those calls.

## Lessons

1. [Interfaces and types](01-interfaces-and-types.md) — Name a shape without a class: `interface`, array types, `Partial` and spread, and the checks only the type checker makes.
2. [Traits](02-traits.md) — Write a contract once, have every impl checked against it, and let operator traits write the metamethods.
3. [Derives and sealed structs](03-derives.md) — Let `@derive` write equality, ordering, cloning and saving from the field list, and let `@sealed` catch typos.
4. [Extension methods](04-extensions.md) — Add methods to `Vector3`, Instance classes and `string` with `export impl`, and see how each call is dispatched.
