# 3. Instances

*Turning state objects into Roblox instances, and reading the world back out again.*

Fusion · Medium

---

This is the half of Fusion that knows Roblox exists. New builds instances and Hydrate adopts ones you already have; special keys parent children, connect events and stream property changes back into state; :set returns its argument so an instance can be captured inline, and attributes work like properties. Everything here takes UsedAs, so a property accepts a constant and a state object equally. The module closes with Project 1, the shop panel.

## Lessons

1. [New and Children](01-new-and-children.md) — Build a whole tree of instances in one expression, and bind any property to a constant or a state object.
2. [Events and properties](02-events-and-properties.md) — OnEvent listens, OnChange watches a property, and Out streams a property back into state.
3. [Hydrate](03-hydrate.md) — Bind Fusion state onto instances somebody already built in Studio, and know what cleanup does to them.
4. [References and attributes](04-refs-and-attributes.md) — Capture instances inline because :set returns its argument, and bind attributes with the same three keys as properties.
