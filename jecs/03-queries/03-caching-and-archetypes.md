# Caching and archetypes

*How archetype storage makes queries fast, when to cache, and what to avoid in hot loops.*

JECS · Queries & Archetypes · 12 min

---

Why JECS? introduced archetypes as "entities with the same components live together". Here is the precise version: it explains why queries are fast and where the costs hide.

## The archetype graph

Every unique *combination* of components is an archetype. A fresh entity starts in the empty **root archetype**; adding `Position` moves it to `[Position]`, adding `Velocity` to `[Position, Velocity]`. Archetypes are connected by cached edges, so a move is a table lookup rather than a search.

When an entity moves, JECS takes it out of the old archetype's entity list, copies its data into the new archetype's columns and updates its record — that copy is why `add` and `remove` cost more than `set`.

## Why queries are fast

Because all entities in an archetype have the same components, a query does not check entities one by one. It matches **archetypes**: does this archetype contain columns for every term? If yes, iterate its rows. Matching is cheap, and the list of matching archetypes is stable enough to cache.

```luau
-- Uncached (default): matches archetypes each time you iterate.
for entity, position, velocity in world:query(Position, Velocity) do end

-- Cached: keeps its list of matching archetypes up to date automatically.
local moving = world:query(Position, Velocity):cached()

for entity, position, velocity in moving do end -- fast; nothing to search
```

|  | Uncached query | Cached query |
| --- | --- | --- |
| Creation cost | Trivial | Higher: the cache is initialized |
| Iteration cost | Match archetypes, then iterate | Iterate prematched archetypes |
| Ongoing cost | None | Archetype creation/deletion updates every cache |
| Best for | Ad-hoc, one-off lookups (e.g. children of a specific parent) | Queries a system runs every frame |

> **An uncached query object is single-pass**
>
> `world:query(A)` is not a reusable handle. It carries the iteration state, so the *second* `for` loop over the same object yields nothing at all — no error, no warning, just an empty loop and a system that silently does nothing.
>
> ```
> local q = world:query(A)
> for e in q do end  -- 3 entities
> for e in q do end  -- 0 entities
> ```
>
> So either write `world:query(A)` fresh at each use site, or `:cached()` it once and keep that. A cached query yields 3 then 3. This bites hardest in the shape it is most tempting to write: a query stored in a module-level local and iterated every frame.

## Fragmentation and churn

Two things quietly degrade an archetype ECS. **Fragmentation**: many distinct component combinations mean many small archetypes, so queries touch more tables — relationships are the usual culprit, because every target of a pair is its own archetype. **Churn**: adding and removing components every frame keeps moving entities around, so a value that flips often belongs *in* a component rather than in a tag. Empty archetypes are not freed automatically; `world:cleanup()` deletes them when you decide it is worth the pause.

```luau
-- Churn: toggles archetype every time the state flips
if grounded then world:add(e, OnGround) else world:remove(e, OnGround) end

-- Cheaper when it flips every frame: keep it as data
world:set(e, Grounded, grounded)
```

Tags are perfect for states that change rarely (Dead, Selected). For per-frame booleans, data is cheaper than moves.

**Key points**

- Archetypes = unique component combinations; adding/removing moves entities, `set` does not.
- Queries match archetypes, not entities; cached queries keep a prematched list.
- Cache per-frame queries once in setup; use uncached queries for ad-hoc lookups.
- Avoid per-frame add/remove churn and needless relationship fragmentation; `world:cleanup()` frees empty archetypes.
