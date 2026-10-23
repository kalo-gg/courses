# 5. Compile-time Alloy

*Attributes and contracts, macros and intrinsics, markup that lowers to your UI library, and tests that vanish from the ship.*

Alloy · Medium

---

Everything in this module happens before the game runs. Attributes put checked metadata on a declaration, and a contract makes the compiler demand the members a framework needs. Macros paste a template at each `$name(…)` call, and the built-in intrinsics see your source text, which no function can. `.alx` files hold markup that lowers to calls on your UI library. The module ends with `@test` functions that live beside the code, vanish from the ship, and become lest specs through `alloy test`.

## Lessons

1. [Attributes and contracts](01-attributes.md) — The built-in attributes, your own with targets and defaults, reading them at run time, and contracts the compiler enforces.
2. [Macros and intrinsics](02-macros.md) — Templates pasted at each `$name(…)` call, their hygiene and limits, and the closed set of built-in intrinsics.
3. [Markup](03-markup.md) — `.alx` files: tags that lower to your UI library, typed attributes, and conditional children that re-run.
4. [Inline tests](04-tests.md) — `@test` functions beside the code, what each artifact does with them, and `alloy test` writing lest specs.
