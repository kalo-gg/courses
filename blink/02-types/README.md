# 2. The Type System

*Every type Blink can send, the ranges that shrink them, and the Luau type each one becomes.*

Blink · Easy

---

Every field in a Blink event has a type, and the type is the whole point: it decides how many bytes go on the wire and what the receiving side refuses. This module walks the numbers and the ranges that bound them, strings and buffers and arrays and what their length prefixes cost, the three table-shaped types, and the two kinds of enum — including the tagged one that stands in for a union Luau will not give you.

## Lessons

1. [Numbers and ranges](01-numbers-and-ranges.md) — Nine number types, and a range that shrinks one of them costs you nothing.
2. [Strings, buffers and arrays](02-strings-and-buffers.md) — Three variable-length types, and the bound you give them picks their length prefix.
3. [Structs, maps and sets](03-structs-and-tables.md) — One fixed shape, one keyed table, one set of flags, and only one of them is free.
4. [Unit enums and tagged enums](04-enums.md) — One byte for a state, and a tagged enum is the union Luau will not give you.
