# 3. Reactive UI

*Bind properties and children to sources, batch writes, and clean up after yourself.*

Vide · Medium

---

With sources, effects and scopes in hand, UI becomes a set of bindings: a property that is a function reruns when its sources change, a child that is a function reparents instances as a list source changes. This module shows both forms, the read and batch helpers that keep components flexible and fast, and cleanup and mount, which tie instance lifetimes to scopes. It ends with your first project: the HUD's inventory panel.

## Lessons

1. [Reactive components](01-reactive-components.md) — Combine a source, an effect and an instance into a component that updates itself.
2. [Implicit effects](02-implicit-effects.md) — A function as a property or child value becomes an effect that keeps it in sync.
3. [read and batch](03-read-and-batch.md) — Accept a value or a source with read; group several writes into one rerun with batch.
4. [cleanup and mount](04-cleanup-and-mount.md) — Tie connections, instances and threads to scopes, and mount a whole app in one call.
