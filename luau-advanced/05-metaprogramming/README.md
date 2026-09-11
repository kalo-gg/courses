# 5. Metamethods in Full

*Operators, proxies, and the readonly tables that make a sandbox possible.*

Luau Advanced · Hard

---

Luau Intermediate used the metamethods that build a class and the read-only view that hides a table. This module covers what is left: the arithmetic and comparison hooks that let a type behave like a number, what the table library does and does not consult, what a proxy costs per access, and the readonly tables that make it possible to hand code to someone without handing them everything.

## Lessons

1. [Making a type do arithmetic](01-operators.md) — The arithmetic and comparison hooks, which operand's metatable runs, and Luau's own rule for __eq.
2. [Callable tables and custom lengths](02-call-and-length.md) — __call makes a table a function, __len redefines #, and the table library ignores both.
3. [Intercepting everything](03-proxies.md) — An empty table plus __index and __newindex sees every access, and charges a function call for each.
4. [Handing over less than everything](04-sandboxing.md) — Frozen tables and proxies are what Luau left you for boundaries, and what neither of them can stop.
