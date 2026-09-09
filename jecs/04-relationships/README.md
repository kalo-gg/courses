# 4. Relationships

*Pairs, hierarchies with ChildOf, wildcards and targets, and cleanup traits that keep the world consistent.*

JECS · Medium

---

Relationships are what set JECS apart: you can link entities to entities (parent/child, ownership, targeting) and query those links directly. This module covers pairs, the built-in ChildOf hierarchy, wildcard queries and world:target, and the cleanup traits and Exclusive trait that decide what happens when a linked entity is deleted. It closes with your first project: a headless wave spawner whose enemies are children of their wave.

## Lessons

1. [Pairs: components made of two ids](01-pairs.md) — Build relationships with jecs.pair, attach data to them, and query for specific links.
2. [Hierarchies with ChildOf](02-hierarchies.md) — The built-in parent/child relationship, walking up and down the tree, and cascading deletes.
3. [Wildcards and targets](03-wildcards-and-targets.md) — Match any target or any relationship, and read the actual targets with world:target.
4. [Cleanup traits and Exclusive](04-cleanup-traits.md) — Decide what happens to linked entities when a component, relationship or target is deleted.
