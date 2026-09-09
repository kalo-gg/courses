# 2. The Syntax

*Elements, attributes, text, fragments and spreads — and the one place reactivity is implicit.*

LuauX · Easy

---

LuauX is JSX adapted to Luau and to Roblox class names. Most of it looks familiar if you have written JSX; the places it differs are worth knowing precisely. Elements and attributes come first, then interpolated text — the only construct that becomes reactive without you asking — then fragments, spreads and comments, and finally renaming, where a project chooses its own vocabulary for classes and properties.

## Lessons

1. [Elements and attributes](01-elements.md) — A Roblox class name, some attributes, children between the tags, and two shorthands that say it once.
2. [Text and interpolation](02-text-and-interpolation.md) — Text between tags, the one construct that becomes reactive on its own, and the attribute that never does.
3. [Fragments, spreads and comments](03-fragments-and-spreads.md) — Several siblings without a wrapper, a props table forwarded in one hole, and notes that never reach the output.
4. [Renaming the vocabulary](04-renaming.md) — Give a project its own names for classes and properties, and learn the two rules that make a rename safe.
