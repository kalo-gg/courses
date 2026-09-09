# 2. Sources, Effects & Scopes

*The reactive core: state in sources, reactions in effects, lifetimes in scopes, and derived values that cache.*

Vide · Easy

---

Vide's UI syntax is a thin layer over a small reactive system that has nothing to do with instances. This module teaches that system on its own: sources that hold values, effects that rerun when the sources they read change, the stable and reactive scopes that decide how long an effect lives, and derived sources that compute and cache. Understand these four ideas and every later feature is a combination of them.

## Lessons

1. [Sources](01-sources.md) — A source stores one value; call it to read, call it with a value to write.
2. [Effects](02-effects.md) — Effects run when the sources they read change; untrack reads a value without subscribing.
3. [Scopes and the reactive graph](03-scopes.md) — Stable scopes hold effects; destroying a root destroys everything created inside it.
4. [Derived sources](04-derived-sources.md) — derive caches a computed value so many readers share one computation and one rerun.
