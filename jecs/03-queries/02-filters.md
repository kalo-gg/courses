# Filters: with and without

*Require or exclude components without pulling their values into the loop, and test one entity against a query.*

JECS · Queries & Archetypes · 9 min

---

Often you need an entity to *have* something without caring about its value, or you need to *exclude* entities that have something. JECS queries have two filter methods for exactly that. Both mutate the query and return it, so `:with(…):without(…)` reads as one chain.

| Method | Meaning | Values yielded? |
| --- | --- | --- |
| `:with(...)` | Entities must also have all of these ids. | No |
| `:without(...)` | Entities must have **none** of these ids. | No |

```luau
local Position = world:component() :: jecs.Id<Vector3>
local Velocity = world:component() :: jecs.Id<Vector3>
local Walking = jecs.tag()
local Frozen = jecs.tag()

-- Moving, walking things that are not frozen.
-- Only Position and Velocity values come out of the loop.
for entity, position, velocity in world:query(Position, Velocity):with(Walking):without(Frozen) do
	world:set(entity, Position, position + velocity * dt)
end
```

Filters keep the loop signature clean: the data you asked for, nothing else.

> **One `:with()` call, however many ids**
>
> Pass every required id to a **single** `:with(…)`. A second call does not add to the first — it replaces it, silently.
>
> `query_with` rebuilds the filter list from scratch on every call and assigns it to `query.filter_with`, so `world:query(A):with(T1):with(T2)` requires only `T2`, and quietly matches entities that have no `T1` at all. Write `world:query(A):with(T1, T2)` instead. (`:without()` has the same shape, so the same rule applies. Mixing the two — `:with(a):without(b)` — is fine, which is why this rarely bites until it does.)

## Common filter recipes

Each recipe below is one sentence of design, not a trick. `:with` says *require this and do not hand it to me*; `:without` says *exclude this*. Reach for them whenever a term would only ever be read as `nil` — a tag, a pair you are filtering on, a state marker — because a filter is both faster to write and clearer than an unused loop variable.

```luau
-- Things that need a Roblox model spawned for them
world:query(ct.Position):without(ct.Model)

-- Living enemies only
world:query(ct.Health):with(ct.Enemy):without(ct.Dead)

-- Everything a player owns that is currently equipped (relationship pairs, Relationships)
world:query(ct.ItemName):with(jecs.pair(ct.OwnedBy, player), ct.Equipped)
```

> **Filters go before caching**
>
> If you cache a query (next lesson), call `:with()` and `:without()` **first**, then `:cached()`. A cached query cannot be refined afterwards.

## Does this entity match?

Sometimes you already hold an entity and only want to know whether a query *would* yield it: is this target still a living enemy? **`query:has(entity)`** answers that without iterating anything, on plain and cached queries alike. It is the cheapest way to write `if enemies:has(target) then` in code that already has the entity in hand, and it respects `:with` and `:without`.

**query:has**

```luau
local living_enemies = world:query(ct.Health):with(ct.Enemy):without(ct.Dead):cached()

if living_enemies:has(target) then
	world:set(target, ct.IncomingDamage, 10)
end
```

**Key points**

- `:with(ids)` requires ids without yielding their values.
- `:without(ids)` excludes entities that have any of the ids.
- Pass every required id to **one** `:with(…)`; a second call replaces the first rather than adding to it.
- `:with(…):without(…)` chains fine, and filters must come before `:cached()`.
- `query:has(entity)` answers "does this entity match?" without iterating.
