# 3. Containers & Big Numbers

*Dictionaries, lists, sets, flags and big numbers: many entries under one name, each validated like a field.*

Scribe · Medium

---

Most player data is a collection: items in a bag, regions unlocked, the last ten runs. This module covers the four container declarators and the element shapes that give every entry the same rules, the caps that turn runaway growth into an error naming the field, flags and big numbers, and the container event family that reports which child moved without ever snapshotting the whole table. It closes with the first project, Emberfall's inventory service, written against the containers you just declared.

## Lessons

1. [Dictionaries](01-dictionaries.md) — DictOf holds many entries of one shape under one name; writing a key creates it and every entry validates like a field.
2. [Arrays and sets](02-arrays-and-sets.md) — ArrayOf for order and eviction, SetOf for membership; which container to pick, and why an index is not an identity.
3. [Flags and big numbers](03-flags-and-big-numbers.md) — Flags hold up to 32 named booleans in one write; Big carries a currency past 2^53 and past 1.8e308.
4. [Container events](04-container-events.md) — Four events, four questions: something changed, which child moved, a key came or went, an entry was inserted or dropped.
