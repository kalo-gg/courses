# 2. Fields & Values

*Declarators give a field a default, a type and rules; accessors are the only way anything reads or writes it.*

Scribe · Easy

---

Every field in a template is declared once and reached the same way afterwards. This module covers the scalar declarators and what a bound, an enum or an optional actually enforces on a write, the accessor surface that Get, Set, Update, Increment and their siblings make up, the two listeners that tell you a value moved and how they differ, and derived fields, which are computed from other fields and refuse every write. By the end, Emberfall's Level follows its Xp with no code that ever sets it.

## Lessons

1. [Declarators and bounds](01-declarators.md) — Int, Number, String, Enum, Optional and Dynamic: what each one enforces on a write, and what a clamp actually does.
2. [Reading and writing values](02-reading-and-writing.md) — Get, Set, Update, Clone and Default cover every field; numbers and booleans add a few more, and Get on a table has a trap.
3. [Observe and Changed](03-listeners.md) — Observe runs now and on every change; Changed only on the transition. Both stop with the function they return.
4. [Derived fields](04-derived-fields.md) — Declare a formula once and read the answer like any other field; nothing ever writes it and nothing can drift.
