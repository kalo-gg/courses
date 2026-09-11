# 3. Generics & Functions

*Typing the functions you already write, then writing ones that work for any type at all.*

Luau Intermediate · Medium

---

A function type says what goes in and what comes out. A generic function says that whatever goes in comes back out, which is how you type the helpers every codebase has: find, map, first. This module covers function types, generics and their explicit instantiation, the type packs that make variadic functions typeable, and the judgement of when to annotate at all. It ends with the typed store.

## Lessons

1. [Typing a function](01-function-types.md) — The arrow type, functions as parameters, and the callback signature that documents itself.
2. [Generics](02-type-parameters.md) — One type parameter turns a function that works for strings into one that works for anything.
3. [Type packs and variadics](03-type-packs.md) — Typing a function that takes any number of arguments, and passing that whole list along.
4. [When not to annotate](04-inference.md) — The checker infers most of it; annotate the boundaries and leave the middle of a function alone.
