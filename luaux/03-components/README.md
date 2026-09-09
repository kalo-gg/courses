# 3. Components

*Any function taking one table — plus the one name that is not yours to use.*

LuauX · Medium

---

A component is a plain Luau function, so there is little to learn and a few things to know precisely: how a tag name is resolved, how props and children reach a component and how a wrapper forwards them, what happens to a child you build conditionally, and why a class with `__call` works at all. The module closes with the first project: the settings panel, built for Vide.

## Lessons

1. [Components are functions](01-functions-as-components.md) — One table in, an element out, indistinguishable from a built-in class at the call site.
2. [Props and children](02-props-and-children.md) — What a component receives, how it defaults a value, and how a wrapper forwards the caller's children.
3. [Children that change](03-dynamic-children.md) — A conditional or a list built once never updates; the lint that catches it and the helpers to reach for.
4. [Class components](04-class-components.md) — A class works as a tag because a tag is a call; put call on the metatable and keep sources, not fields.
