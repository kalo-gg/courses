# 1. Types in Luau

*What a type checker buys you, how to switch it on, and how to read what it tells you.*

Luau Intermediate · Easy

---

Luau's type checker is optional, which is why most people never turn it on and never find out what it does for them. This module makes the case with a bug it catches, shows how to annotate a variable and a function, explains the three modes and which to pick, and spends a whole lesson on reading the errors, because an unfamiliar type error is worse than no type error at all.

## Lessons

1. [Why a type checker](01-why-types.md) — A mistake caught while you type costs seconds; the same mistake found at run time costs an hour.
2. [Annotating values and functions](02-annotations.md) — The colon syntax on locals, parameters and returns, and the cast operator for the rest.
3. [nocheck, nonstrict and strict](03-modes.md) — Three comments at the top of a file decide how hard the checker looks, and the default is nonstrict.
4. [Reading a type error](04-reading-errors.md) — Expected against got, where each side came from, and the three errors you will see most.
