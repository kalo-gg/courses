# 4. Backends

*One file, four libraries — and a precise line between what a setting can change and what it cannot.*

LuauX · Medium

---

LuauX lowers to React, Vide, Fusion and Fluid without knowing about any of them, by separating the **arrangement** of the constructor call, which is a backend, from the **contents** of the props table, which is configuration. Two arrangements, the `[factory]` block, what ports between libraries and what never did, and how a whole screen is assembled from component files, style tables and an entry script.

## Lessons

1. [Two arrangements](01-arrangements.md) — Four libraries, two shapes of constructor call, and why a runtime adapter was rejected.
2. [The [factory] block](02-factory.md) — Five or six lines that retarget the whole project, and the rule that writing them turns every default off.
3. [What ports, and what does not](03-porting.md) — The markup travels between libraries; children forwarding, Instance reads and React bindings do not.
4. [Assembling a screen](04-assembling-a-screen.md) — One component per file, a style module, an App that assembles them and a LocalScript that mounts: the real layout.
