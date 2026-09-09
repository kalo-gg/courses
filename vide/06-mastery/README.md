# 6. Shipping a Vide App

*Mount into PlayerGui, debug with strict mode, follow the patterns that scale, test a screen, and build the complete HUD.*

Vide · Hard

---

Everything so far ran in a root. A real place needs an entry point, a habit of catching mistakes early, and a few conventions so components stay readable as the UI grows. This module mounts an app from a LocalScript, turns on strict mode and reads its errors, collects the patterns the Vide community settled on, plans the HUD and tests a component headlessly, and finishes with the project: the HUD, grown from the panel you built in Reactive UI.

## Lessons

1. [Mounting an app](01-mounting.md) — Where the code lives, how it starts, and how a whole UI tree hangs off one root.
2. [Strict mode and debugging](02-strict-mode.md) — Let Vide double-check your reactive code, and recognize its common error messages.
3. [Patterns that scale](03-patterns.md) — Conventions for props, state ownership, lists and performance from real Vide projects.
4. [Planning and testing the HUD](04-planning-the-hud.md) — Split a screen into a state module, components and an entry script, then test a component without Roblox.
