# 5. Hooks, Signals & Observers

*React to component changes the right way: hooks, multi-listener signals, observers, and when not to use them.*

JECS · Medium

---

Sometimes something must happen the moment data changes: destroy a Model when its component is removed, stamp an attribute when one is added. This module covers JECS's three hooks, the multi-listener signals added in 0.9, the OB observer/monitor module, and the design guidance from the JECS author on keeping gameplay logic in systems.

## Lessons

1. [Component hooks](01-hooks.md) — OnAdd, OnChange and OnRemove: one callback per component, with precise timing rules.
2. [Signals: many listeners](02-signals.md) — world:added, world:changed and world:removed with unsubscribe functions.
3. [Observers and monitors](03-observers-and-monitors.md) — React when an entity starts or stops matching a whole query, using the OB module shipped with JECS.
4. [Hooks are not systems](04-hooks-vs-systems.md) — Design guidance from the JECS author on where logic lives, and a refactor that moves gameplay out of a hook.
