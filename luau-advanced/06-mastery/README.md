# 6. Shipping Fast Luau

*Allocation, hot loops, the limits the VM really has, and an API that stays fast.*

Luau Advanced · Hard

---

The last module is about restraint: knowing which of the previous five modules' tools to reach for, and how often the answer is none of them. Allocation, the shape of a hot loop, the limits the VM genuinely has including the absence of tail calls, and the API decisions that make a fast implementation possible for whoever calls it. Then the whole simulation.

## Lessons

1. [The cost you cannot see](01-allocation.md) — Tables, closures and strings allocate; the collector charges for them later, and elsewhere.
2. [The shape of a hot loop](02-hot-loops.md) — Hoist the lookups, allocate nothing, flatten the data, and leave the cold code alone.
3. [The limits the VM really has](03-limits.md) — 200 locals, 200 upvalues, 255 registers, a bounded stack, and no tail calls at all.
4. [An API that stays fast](04-fast-apis.md) — Return what exists, take an output parameter on hot paths, and say in the name when something mutates.
