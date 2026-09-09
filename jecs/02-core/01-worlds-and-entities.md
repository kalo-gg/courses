# Worlds and entities

*Creating a world, spawning and deleting entities, and what an entity id really contains.*

JECS · Entities, Components & Systems · 10 min

---

A **world** owns every entity, component and archetype. You create one with `jecs.world()` and pass it around (or export it from a shared module). Almost every JECS call is a method on the world.

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)

local world = jecs.world()

-- During development, enable DEBUG mode for extra runtime checks:
-- local world = jecs.world(true)
```

`jecs.world(true)` turns on checks for stale entity handles, unregistered ids and misuse of wildcard pairs — on `add`, `set` and `remove` **only**. `get`, `has`, `query` and `delete` keep their unchecked bindings, so a `world:get` on a dead handle still returns quietly. It costs performance, so leave it off in production.

## Spawning and deleting

```luau
local entity = world:entity() -- a fresh id with no components
print(entity)                 -- e.g. 272 (lower ids are reserved for built-ins and components)

world:delete(entity)          -- removes every component, then frees the id

print(world:contains(entity)) -- false: the handle is no longer alive
```

`world:delete` does more than free the id. It also runs **cleanup traits** (Relationships) so that nothing else in the world keeps pointing at a dead entity, and it calls any `OnRemove` hooks (Hooks, Signals & Observers) for the components being removed. For now, remember: *delete is the only way an entity stops existing*. Removing all of its components leaves an empty but alive entity.

**Emptying an entity without killing it**

```luau
local entity = world:entity()
world:set(entity, Position, Vector3.zero)
world:add(entity, Enemy)

world:clear(entity)           -- strips every component and pair

print(world:contains(entity)) -- true: still alive, just empty
print(world:get(entity, Position)) -- nil
```

`world:clear(entity)` is the middle ground between `world:remove` (one id) and `world:delete` (the whole entity). `OnRemove` hooks fire for everything it strips — with `delete` unset, since the entity is not going anywhere — and the handle stays valid, so a pooled entity can be reset and reused instead of deleted and reallocated, and any long-lived reference to it keeps working.

## What is inside an entity id?

An entity id is a plain Luau number, but it packs two pieces of information: a 24-bit **index** and a 24-bit **generation**.

When you delete an entity its index goes back into a free list. The next `world:entity()` call may *reuse that index*, but with the generation incremented. That means an old handle you accidentally kept around will not match the new entity: `world:contains(old)` returns `false` even though the index is in use again.

```luau
local a = world:entity()
world:delete(a)

local b = world:entity() -- reuses a's index, generation + 1

print(jecs.ECS_ID(a) == jecs.ECS_ID(b))               -- true: same index
print(jecs.ECS_GENERATION(a), jecs.ECS_GENERATION(b)) -- 0  1
print(world:contains(a), world:contains(b))           -- false  true
```

`jecs.ECS_ID` and `jecs.ECS_GENERATION` decode the two halves. You rarely need them, but they explain why recycled ids look like huge numbers.

## contains vs exists

| Method | Returns `true` when… | Typical use |
| --- | --- | --- |
| `world:contains(e)` | the entity is **alive**: its index is in use *and* the generation matches. | Validating a handle before using it. |
| `world:exists(e)` | the index slot is occupied, **ignoring generation**. | Replication code that maps ids from another world (Roblox Integration). |

You can also create an entity with a **specific id** using `world:entity(id)`. It skips the range check, fills the sparse array up to that id and raises the allocator's `max_id` to match — which is fine in a world that allocates nothing of its own, and a trap in one that does. Mirroring on the client explains why replication maps foreign ids through a table instead of adopting them with this call.

**Key points**

- `jecs.world()` creates the world; `jecs.world(true)` adds DEBUG checks for development.
- `world:entity()` spawns, `world:delete(e)` destroys (and runs cleanup + hooks).
- Ids pack an index and a generation, so stale handles are detected by `world:contains`.
- `world:exists` ignores generation; `world:entity(id)` creates an entity with a chosen id.
