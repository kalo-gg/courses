# Components and data

*set, get, has and remove; typing components; and the fact that components are entities too.*

JECS · Entities, Components & Systems · 11 min

---

A component is a **key to a storage column**. Think of the world as a spreadsheet: entities are rows, components are columns and the cell is the value. `world:component()` registers a new column and returns its id.

```luau
local Position = world:component() :: jecs.Id<Vector3>
local Health = world:component() :: jecs.Id<number>
local Inventory = world:component() :: jecs.Id<{ items: { string }, gold: number }>
```

The cast to `jecs.Id<T>` is the official way to give a component a type. Luau cannot infer it, so the cast is how `set`/`get`/`query` become type-safe.

## The four verbs

```luau
local e = world:entity()

-- set: adds the component if missing, otherwise updates the value
world:set(e, Position, Vector3.new(10, 20, 30))
world:set(e, Health, 100)
world:set(e, Health, 50) -- update in place

-- get: returns the value, or nil if the entity lacks the component
local hp = world:get(e, Health)     -- number?
print(hp)                          -- 50

-- has: boolean check without reading the value
print(world:has(e, Position))      -- true

-- remove: takes the component away (idempotent)
world:remove(e, Health)
print(world:get(e, Health))        -- nil
world:remove(e, Health)            -- safe: does nothing
```

> **get returns T?**
>
> `world:get` always returns an optional. The entity might have been deleted, or might simply not have that component. Handle `nil` explicitly, or use `assert(world:get(e, Position))` when you *know* it must exist and want a loud failure otherwise.

`get` and `has` accept **up to four components at once**, which avoids repeated lookups:

```luau
local pos, hp = world:get(e, Position, Health)  -- (Vector3?, number?)

if world:has(e, Position, Health) then          -- true only if it has BOTH
	-- ...
end
```

Passing a fifth component errors with `args exceeded`. Use a query (Queries & Archetypes) when you need more.

## Components are entities

Here is the idea that makes JECS (and flecs) so uniform: **a component id is itself an entity**. `world:component()` creates an entity and adds the built-in `jecs.Component` trait to it. Because components are entities, everything that works on entities works on components: you can attach metadata to them, query them, even relate them to each other.

The most useful metadata is the built-in `jecs.Name` component, which debuggers like Jabby display:

```luau
world:set(Position, jecs.Name, "Position")

print(world:get(Position, jecs.Name))          -- "Position"
print(world:has(Position, jecs.Component))     -- true
```

> **The 256 component limit**
>
> Component ids created with `world:component()` occupy the range 1 to 256 so they can be stored compactly. That is plenty for most games. If you ever need more, create a normal entity and add the trait yourself: `local id = world:entity(); world:add(id, jecs.Component)`.

**Key points**

- `world:component() :: jecs.Id<T>` registers a typed column.
- `set` adds or updates; `get` returns `T?`; `has` checks; `remove` is idempotent.
- `get` and `has` take up to four components at once.
- Component ids are entities: attach `jecs.Name` and other metadata with `set`.
