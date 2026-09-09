# Tags, singletons and preregistration

*Zero-data components, world-level resources, and declaring ids before the world exists.*

JECS · Entities, Components & Systems · 10 min

---

Not every component needs data. `Dead`, `Enemy`, `Selected`, `Networked`: these are **tags**, markers whose only meaning is *present or absent*. A tag takes zero bytes of storage per entity and tells readers of your code that there is nothing to read or write.

Any plain entity works as a tag at runtime too (`local Dead = world:entity()`); the exercises on this site use that form because the sandbox creates the world for you.

```luau
local Dead = jecs.tag()      -- preregistered tag (recommended)
local world = jecs.world()   -- create the world AFTER declaring tags/components

local e = world:entity()
world:add(e, Dead)           -- tags are added with add, never set
print(world:has(e, Dead))    -- true
world:remove(e, Dead)
print(world:has(e, Dead))    -- false

print(jecs.is_tag(world, Dead)) -- true
```

A tag is simply an entity id *without* the `jecs.Component` trait. You cannot `world:set` a tag because there is no column to store data in.

## Preregistration

You can declare components and tags **before the world exists** using `jecs.component()` and `jecs.tag()`. When `jecs.world()` runs, every preregistered id is allocated in order. This is the recommended style for a real project because it lets a shared `components.luau` module define ids without needing a world instance, and it keeps the ids stable.

The one rule: **declare everything before calling `jecs.world()`**. Ids registered after the world already exists are not applied to it (in DEBUG mode you get an error telling you exactly that).

**components.luau (shared module)**

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)

local Position = jecs.component() :: jecs.Id<Vector3>
local Velocity = jecs.component() :: jecs.Id<Vector3>
local Dead = jecs.tag()

-- Metadata can be forward-declared too, since there is no world yet:
jecs.meta(Position, jecs.Name, "Position")
jecs.meta(Velocity, jecs.Name, "Velocity")
jecs.meta(Dead, jecs.Name, "Dead")

return {
	Position = Position,
	Velocity = Velocity,
	Dead = Dead,
}
```

**world.luau**

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)
require(ReplicatedStorage.Shared.components) -- make sure ids are declared first

return jecs.world() -- preregistered ids are allocated here
```

## Singletons

Sometimes there is exactly one of something: the current match state, a time-of-day value, a reference to a physics helper. JECS has no separate "resource" API for that. Instead it uses the fact that components are entities: **store the value on the component's own id**.

```luau
local TimeOfDay = world:component() :: jecs.Id<number>

world:set(TimeOfDay, TimeOfDay, 5)         -- the component holds itself
local t = world:get(TimeOfDay, TimeOfDay)  -- 5
print(`The time of day is {t}`)
```

A singleton is a component set on itself. It is accessible anywhere you have the world, with no global variables.

**Key points**

- Tags are ids without data: `jecs.tag()`, then `world:add` / `world:has` / `world:remove`.
- Preregister with `jecs.component()`, `jecs.tag()` and `jecs.meta()` **before** `jecs.world()`.
- A singleton is a component set on its own id: `world:set(T, T, value)`.
- Keep components in one shared module so server and client agree on ids.
