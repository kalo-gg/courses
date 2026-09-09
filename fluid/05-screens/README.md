# 5. Building Real Screens

*Components are functions, screens are mounted, and the graph tells you when you got it wrong.*

Fluid · Medium

---

Fluid has no component system, so a component is a function. This module puts that convention to work in a real place: mounting a tree into a ScreenGui that already exists, binding onto instances built in Studio, tagging them so other systems can find them, and reading the graph's error messages closely enough to find the scope you forgot.

## Lessons

1. [Components are functions](01-components.md) — There is no component API: a component is a function that returns what create returned.
2. [Mounting a screen](02-mounting.md) — Bind a component onto a ScreenGui that already exists, and know what unmounting does not do.
3. [Instances you did not build](03-existing-instances.md) — Bind onto a Studio template, read a property back, reach the instance itself, and tag it.
4. [Debugging the graph](04-debugging-the-graph.md) — Read Fluid's messages closely enough to find the scope you forgot or the yield you did not mean.
