# 6. Best Practices & Capstone

*Patterns that scale, a reactivity checklist, reading the output when it goes wrong — and the finished screen.*

LuauX · Hard

---

The habits that keep a LuauX project readable a year on: style tables and dotted namespaces, props that accept a value or a source, the checklist of what updates and what never will, and how to read compiled output when Studio points at a line. An architecture lesson plans the final project, and the project ships the settings screen: Rojo wiring, a keybind list, a confirm dialog, and a port to a second backend.

## Lessons

1. [Patterns that scale](01-patterns.md) — Style tables, dotted namespaces and props that take a value or a source: the habits a screen needs.
2. [What updates, and what never will](02-reactivity-checklist.md) — Where a source may be read, which reads survive the first frame, and the checklist for when nothing moves.
3. [Reading the output](03-debugging-the-lowering.md) — Open the compiled file at the same line number and read what the compiler actually emitted.
4. [Planning the settings screen](04-architecture.md) — Draw the file tree, decide where state lives, and write the config before the first tag.
