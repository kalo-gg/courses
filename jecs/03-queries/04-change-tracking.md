# Change tracking with queries

*Detect added, changed and removed components using only queries and a Previous pair.*

JECS · Queries & Archetypes · 10 min

---

A common need: run code only for entities whose `Position` was **added** this frame, or **changed** since last frame, or **removed**. JECS offers hooks and signals for that (Hooks, Signals & Observers), but there is a purely query-based technique from the official examples that is worth knowing. It uses a relationship pair as a "previous value" slot and needs no callbacks at all.

**Three cached queries**

```luau
local pair = jecs.pair
local Position = world:component() :: jecs.Id<Vector3>
local Previous = world:entity() -- any plain entity can be the relation
                                -- (the official example uses the built-in jecs.Rest)

local added = world
	:query(Position)
	:without(pair(Previous, Position))
	:cached()

local changed = world
	:query(Position, pair(Previous, Position))
	:cached()

local removed = world
	:query(pair(Previous, Position))
	:without(Position)
	:cached()
```

`pair(Previous, Position)` is a component that stores a *copy* of the last Position we processed. Its data type comes from Position because `Previous` has no data.

> **`jecs.Rest`, if you read the upstream example**
>
> Any entity works as the relation, and a named local like `Previous` says what it is for. But `examples/queries/changetracking.luau` writes `local Previous = jecs.Rest` — a built-in id JECS exports for exactly this sort of marker relation, so you do not have to allocate one.
>
> The two are interchangeable here. Use whichever you like; just recognize `jecs.Rest` when you meet it, because the upstream example gives no clue what it is.

Read the three queries in English:

- **added**: has `Position` but no previous copy yet.
- **changed**: has both, so we can compare new and old.
- **removed**: has a previous copy but no longer has `Position`.

**Processing each set**

```luau
for entity, position in added do
	print(`Added {entity}: {position}`)
	world:set(entity, pair(Previous, Position), position)
end

for entity, new, old in changed do
	if new ~= old then
		print(`{entity} moved from {old} to {new}`)
		world:set(entity, pair(Previous, Position), new)
	end
end

for entity in removed do
	print(`Position was removed from {entity}`)
	world:remove(entity, pair(Previous, Position))
end
```

Each loop writes the marker so the entity leaves that set until the next real change.

## Modifying entities while iterating

Notice that the loops above add and remove components on the entity being iterated. JECS iterates each archetype **backwards**, which is exactly why removing or deleting the *current* entity during iteration is safe. The official example uses this pattern, and the test suite covers deleting while iterating.

The docs make no promise about adding components that move the current entity into another archetype the same query visits. Keep mutations to the entity in hand, as the marker pattern does.

> **When to use which**
>
> This query-based approach batches change handling into a predictable point in the frame and needs no callbacks. Hooks and signals (Hooks, Signals & Observers) react *immediately* when data changes and are better for enforcing invariants such as destroying a Model. Both are legitimate; pick per use case.

**Key points**

- A `pair(Previous, Component)` marker turns plain queries into added/changed/removed sets.
- Write the marker inside each loop so entities leave the set until they change again.
- Removing/deleting the current entity during iteration is safe (JECS iterates backwards).
- Use this for batched, frame-ordered change handling; use hooks/signals for immediate reactions.
