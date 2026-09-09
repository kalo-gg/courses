# Pairs: components made of two ids

*Build relationships with jecs.pair, attach data to them, and query for specific links.*

JECS · Relationships · 10 min

---

A **pair** is a single id that encodes two entities: a *relationship* (the first element, such as `Likes` or `Owns`) and a *target* (the second, such as `alice` or `car`). Because a pair is just an id, you add it, set it, query it and remove it exactly like a component.

```luau
local pair = jecs.pair

local Likes = world:entity()                                   -- relationship with no data
local Owns = world:component() :: jecs.Id<{ since: number }>  -- relationship with data
local alice = world:entity()
local car = world:entity()
local bob = world:entity()

world:add(bob, pair(Likes, alice))
world:set(bob, pair(Owns, car), { since = 2024 })

print(world:has(bob, pair(Likes, alice)))       -- true
print(assert(world:get(bob, pair(Owns, car))).since) -- 2024
```

Where does the *data type* of a pair come from? From the **relationship** if it is a component (`Owns` above). If the relationship is a plain entity or tag, the data comes from the **target** instead, which is how `pair(Previous, Position)` in Change tracking with queries stored a `Vector3`. If neither has data, the pair is a tag.

## Querying pairs

A pair is an ordinary id, so everything you already know applies to it: `world:add`, `world:set`, `world:has`, `world:remove` and query terms all take `pair(Rel, target)` in the slot a component would occupy. What changes is *where* it lives — the pair goes on the entity at the near end of the link, so `OwnedBy` sits on the sword, not on the player.

```luau
-- Everyone who likes alice
for entity in world:query(pair(Likes, alice)) do
	print(`{entity} likes alice`)
end

-- Everyone who owns the car, with the ownership record
for entity, ownership in world:query(pair(Owns, car)) do
	print(`{entity} has owned the car since {ownership.since}`)
end

-- Pairs combine with normal terms and filters
for entity, position in world:query(Position):with(pair(Likes, alice)) do end
```

> **Inspecting a pair id**
>
> `jecs.IS_PAIR(id)` tells you whether an id is a pair. `jecs.pair_first(world, p)` and `jecs.pair_second(world, p)` recover the two *alive* entities. In everyday code you will use `world:target` (next lessons) instead, but these are handy in debugging and serialization code.

**Key points**

- `jecs.pair(relationship, target)` makes an id you can add/set/has/remove/query like a component.
- Data type comes from the relationship if it is a component, otherwise from the target.
- `world:query(pair(Rel, target))` finds every entity with that exact link.
