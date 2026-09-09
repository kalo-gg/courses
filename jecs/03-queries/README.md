# 3. Queries & Archetypes

*Find entities fast: query terms, with/without filters, caching, and how archetypes shape performance.*

JECS · Easy

---

Queries are how systems find the entities they care about. This module covers the query API from the basic `world:query(A, B)` loop through `with`/`without` filters and cached queries, then looks under the hood at archetypes so you can make good performance decisions and track changes without hooks.

## Lessons

1. [Query basics](01-query-basics.md) — Iterating entities that have a set of components, and what each loop variable holds.
2. [Filters: with and without](02-filters.md) — Require or exclude components without pulling their values into the loop, and test one entity against a query.
3. [Caching and archetypes](03-caching-and-archetypes.md) — How archetype storage makes queries fast, when to cache, and what to avoid in hot loops.
4. [Change tracking with queries](04-change-tracking.md) — Detect added, changed and removed components using only queries and a Previous pair.
