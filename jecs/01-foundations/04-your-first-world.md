# Your first world

*Create a world, declare a typed component, spawn an entity, set a value and read it back.*

JECS · ECS Foundations · 10 min

---

Every JECS program is the same five calls: create a **world**, declare a **component**, create an **entity**, `set` a value, `get` it back. The script below is the whole thing. Read it once, then walk through it line by line.

**hello.server.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local jecs = require(ReplicatedStorage.Packages.jecs)

-- A world stores entities and their components.
local world = jecs.world()

-- Declare a component and its data type.
local Position = world:component() :: jecs.Id<Vector3>

-- Create an empty entity and give it data.
local entity = world:entity()
world:set(entity, Position, Vector3.new(1, 2, 3))

-- Read it back. get() returns Vector3? because the entity might not have it.
print(`Entity's Position is {world:get(entity, Position)}`)
```

Walk through it line by line:

1. `jecs.world()` creates a **world**, the container for everything.
2. `world:component()` registers a new component id. The cast `:: jecs.Id<Vector3>` tells the type checker what the component holds.
3. `world:entity()` creates an entity: an id with no data.
4. `world:set` attaches data. If the entity does not have the component yet it is added; otherwise the value is updated.
5. `world:get` reads data back and returns `nil` if the entity does not have that component.

## Reading it back safely

**get returns T?**

```luau
local position = world:get(entity, Position)
if position then
	print(`at {position}`)
else
	print("no position")
end

-- when it must exist and silence would be a bug:
local hp = assert(world:get(entity, Health), "hero has no Health")
```

`world:set` is *add-or-update*: the first call attaches the component, later calls change the value. `world:get` is the mirror image and always returns an optional. You will meet the other verbs (`has`, `add`, `remove`) in the next module; these two carry most everyday code.

> **Running JECS outside Roblox**
>
> Because JECS is pure Luau, you can run scripts with the `luau` CLI or Lune for quick experiments and unit tests. Outside Roblox there is no `Vector3`, so the official tutorials use the built-in `vector` library (`vector.create(1, 2, 3)`). Inside Roblox, `Vector3` values *are* native vectors, so either spelling works there.

**Key points**

- `jecs.world()` creates the world; `world:component()` and `world:entity()` create ids.
- `world:set` adds or updates data; `world:get` returns the value or `nil`.
- Type components with `:: jecs.Id<T>` so the type checker can help you.
- Outside Roblox use `vector.create`; inside, `Vector3` values are native vectors.
