# 2. The Reactive Graph

*Derived values, effects, ownership, and a node whose value has not arrived yet.*

Fluid · Easy

---

Sources are only the start. This module walks the rest of the graph: derived values and how to read one without subscribing, where side effects belong and why none of them may yield, how teardown is attached to the scope that owns it — and async nodes, the one place in Fluid a callback is allowed to wait.

## Lessons

1. [Derived values](01-deriving-values.md) — Cached values that recompute only when read, and only when the answer actually changed.
2. [Effects](02-effects.md) — Run something when state changes, know what you subscribed to, and never yield inside one.
3. [Cleanup and ownership](03-cleanup.md) — A root tears down the graph it owns; your instances only go if you hand them over.
4. [Async nodes](04-async.md) — A reactive node whose value arrives later, and a state that only ever says busy or ok.
