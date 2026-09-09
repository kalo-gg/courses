# 2. State Objects

*Values you write to, values derived from them, and the one place side effects belong.*

Fusion · Easy

---

Fusion has three state pieces and they divide cleanly. A Value is what you set. A Computed is worked out from other state and must stay pure. An Observer is the escape hatch for side effects. This module builds each one, shows why reading with use differs from reading with peek, and finishes on the scope every callback gets, the inner scopes you can make with innerScope, and the warning Fusion prints when two lifetimes are the wrong way round.

## Lessons

1. [Values](01-values.md) — scope:Value(x) holds state; :set writes it, peek reads it, and a write only fires when the value changed.
2. [Computeds](02-computeds.md) — Work a value out from other state with use, and keep the calculation pure and immediate.
3. [Observers](03-observers.md) — The one place a change is allowed to do something: onChange for later changes, onBind to run now as well.
4. [Inner scopes](04-inner-scopes.md) — The scope every callback is handed, the inner scopes you make with innerScope, and the warning when lifetimes cross.
