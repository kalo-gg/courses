# What is an ECS?

*Entities, components and systems, and how they differ from Instance scripts and OOP.*

JECS · ECS Foundations · 9 min

---

Most Roblox code you have seen attaches *behavior* to *things*: a Script inside a Part, a `Zombie` class with a `:Attack()` method, a ModuleScript per enemy type. That works for small projects, but as a game grows the things start to multiply (flying zombies, frozen zombies, zombies that are also bosses) and every new combination means another class or another copy-pasted script.

An **Entity Component System** (ECS) turns that around. Instead of objects that own behavior, you have three separate ingredients.

## The three ingredients

| Ingredient | What it is | Example |
| --- | --- | --- |
| **Entity** | A unique id. Nothing more: no fields, no methods. | `42`, the zombie |
| **Component** | Plain data attached to an entity. One component = one kind of data. | `Position = Vector3`, `Health = 100`, `Zombie` (a tag with no data) |
| **System** | A function that runs over every entity that has a particular set of components. | *movement*: for everything with `Position` and `Velocity`, move it |

An entity is like a row in a spreadsheet, a component is a column, and a system is a formula that runs down every row that has values in the columns it cares about. Entities do not know what systems exist, and systems do not know what "kinds" of entity exist. They only see data.

**The same zombie as a class**

```luau
local Zombie = {}
Zombie.__index = Zombie
function Zombie.new(position) return setmetatable({ position = position, health = 100 }, Zombie) end
function Zombie:step(dt) self.position += self.velocity * dt end
-- FlyingZombie, FrozenZombie, BossZombie... each one a new class
```

Behavior and data are welded together. Every new combination is a new class.

**A first taste (JECS syntax)**

```luau
local world = jecs.world()

-- Components: named columns of data
local Position = world:component() :: jecs.Id<Vector3>
local Velocity = world:component() :: jecs.Id<Vector3>

-- An entity: just an id, then we attach data to it
local zombie = world:entity()
world:set(zombie, Position, Vector3.new(0, 5, 0))
world:set(zombie, Velocity, Vector3.new(0, 0, -4))

-- A system: a plain function over matching entities
local function movement(dt: number)
	for entity, position, velocity in world:query(Position, Velocity) do
		world:set(entity, Position, position + velocity * dt)
	end
end
```

Do not worry about every detail yet. The shape is what matters: data on entities, logic in systems.

> **Composition, not inheritance**
>
> The zombie is not an instance of a `Zombie` class. It is an entity that *happens to have* `Position`, `Velocity` and a `Zombie` tag. Want a flying zombie? Add a `Flying` tag and let the movement system handle gravity differently. No subclass required.

## Why games use this pattern

- **Decoupling.** A `damage` system does not care whether the thing it damages is a zombie, a crate or a player. If it has `Health`, it can be damaged.
- **Composition.** New behavior is a new component plus a system, not a new class hierarchy.
- **Performance.** Storing components in tight arrays and processing them in batches is far more cache-friendly than chasing pointers between objects. Archetype ECSs like JECS take this idea furthest.

ECS is not a silver bullet. For a tiny project it adds ceremony, and Roblox Instances do not go away: an ECS manages your *game state*, and Instances remain how the world is rendered. Keeping the two in sync is what Roblox Integration is about.

**Key points**

- An **entity** is just a unique id.
- A **component** is one kind of plain data attached to an entity.
- A **system** is a function that runs over every entity with a given set of components.
- Behavior comes from composition (which components an entity has), not inheritance.
