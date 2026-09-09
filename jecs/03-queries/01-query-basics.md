# Query basics

*Iterating entities that have a set of components, and what each loop variable holds.*

JECS · Queries & Archetypes · 9 min

---

A query selects every entity that has **all** of the listed components, and hands you their values in the same order. It is the bread and butter of every system.

```luau
local Position = world:component() :: jecs.Id<Vector3>
local Velocity = world:component() :: jecs.Id<Vector3>
local Mass = world:component() :: jecs.Id<number>

local e1 = world:entity()
world:set(e1, Position, Vector3.zero)

local e2 = world:entity()
world:set(e2, Position, Vector3.zero)
world:set(e2, Velocity, Vector3.one)

local e3 = world:entity()
world:set(e3, Position, Vector3.zero)
world:set(e3, Velocity, Vector3.one)
world:set(e3, Mass, 10)

-- Matches e2 and e3 (both have Position AND Velocity). e1 lacks Velocity.
for entity, position, velocity in world:query(Position, Velocity) do
	print(`{entity} has Position {position} and Velocity {velocity}`)
end
```

The loop variables are: the **entity id** first, then one value per query term, in order. Extra components an entity has (like `Mass` on `e3`) do not matter; the entity just needs to have *at least* the ones you asked for.

## Tags and ids in queries

Any id can be a query term: components with data, tags, relationship pairs, even plain entities. Terms without data simply yield `nil` in their slot, so the idiomatic way to require a tag is to list it *after* the data components or, better, use the `:with()` filter from the next lesson.

```luau
local Enemy = jecs.tag() -- declared before the world in a real project

-- Every enemy with a position. The Enemy slot yields nil, so we ignore it.
for entity, position in world:query(Position, Enemy) do
	print(`enemy {entity} at {position}`)
end

-- Just the entities that have a given id, no data at all:
for entity in world:each(Enemy) do
	print(`{entity} is an enemy`)
end
```

`world:each(id)` is a lightweight iterator over the entities that have one id. It is handy for tags and singletons.

## Reading vs writing inside a query

Writing back with `world:set` inside the loop is normal and, because the component already exists, cheap. For table-valued components you may also mutate the table in place (`pos.x += 1`) when nobody needs to be *notified* of the change. Hooks, Signals & Observers explains why `set` matters when hooks or signals are involved.

```luau
for entity, position, velocity in world:query(Position, Velocity) do
	world:set(entity, Position, position + velocity * dt)
end
```

**Key points**

- `world:query(A, B)` yields `entity, a, b` for every entity that has **all** terms.
- Extra components never exclude an entity.
- Tags and other data-less ids yield `nil` in their slot; `world:each(id)` iterates a single id.
- Writing with `world:set` inside the loop is normal and cheap for existing components.
