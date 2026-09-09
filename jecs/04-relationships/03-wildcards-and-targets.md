# Wildcards and targets

*Match any target or any relationship, and read the actual targets with world:target.*

JECS · Relationships · 9 min

---

Sometimes you do not know the target. "Who eats *anything*?" or "what has *any* relationship with alice?" The `jecs.Wildcard` entity (alias `jecs.w`) stands in for either half of a pair.

| Pattern | Matches |
| --- | --- |
| `pair(Eats, jecs.Wildcard)` | Entities that eat *something* (any target) |
| `pair(jecs.Wildcard, alice)` | Entities with *any* relationship whose target is alice |

```luau
local pair = jecs.pair
local Eats = world:entity()
local Apples = world:entity()
local Oranges = world:entity()
local bob = world:entity()

world:add(bob, pair(Eats, Apples))
world:add(bob, pair(Eats, Oranges))

print(world:has(bob, pair(Eats, jecs.Wildcard))) -- true

for entity in world:query(pair(Eats, jecs.Wildcard)) do
	local food = world:target(entity, Eats) -- first target, index 0
	print(`{entity} eats {food}`)
end
```

## world:target

A wildcard query tells you *that* an entity has a matching pair, but JECS only supports the *Any* wildcard, so it yields each entity **once** and only the first matching pair. To read the actual targets use `world:target(entity, relationship, index)`. The index is **0-based** and the call returns `nil` once you run past the last target, so a `while` loop walks all of them.

```luau
print(world:target(bob, Eats, 0) == Apples)   -- true
print(world:target(bob, Eats, 1) == Oranges)  -- true
print(world:target(bob, Eats, 2))             -- nil
print(world:target(bob, Eats) == Apples)      -- index defaults to 0

local nth = 0
local food = world:target(bob, Eats, nth)
while food do
	print(`bob eats {food}`)
	nth += 1
	food = world:target(bob, Eats, nth)
end
```

> **Wildcards are for reading, not writing**
>
> You cannot `add`, `set` or `remove` a wildcard pair; in DEBUG mode JECS errors with a message saying so. To remove every `(Eats, *)` from an entity, walk the targets with `world:target` and remove each concrete pair.

On the unreleased `main` branch there is also `world:targets(entity, relationship)`, an iterator over all targets. It is not in 0.11.0, so stick with the `world:target` loop until a release includes it.

**Key points**

- `pair(Rel, jecs.Wildcard)` = any target; `pair(jecs.Wildcard, target)` = any relationship.
- Wildcard queries yield an entity once; `world:target(e, Rel, nth)` (0-based, `nil` at the end) enumerates targets.
- Never add/set/remove a wildcard pair.
