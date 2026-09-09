# Why JECS?

*Archetype storage, relationships and the features that set JECS apart.*

JECS · ECS Foundations · 9 min

---

**JECS** is an ECS written in pure Luau by Marcus (Ukendio), inspired by [flecs](https://github.com/SanderMertens/flecs), a widely used C ECS. It runs inside Roblox and in standalone Luau (the `luau` CLI or Lune), which is why the official tutorials are plain `.luau` files.

This course tracks **JECS 0.11.0** (March 2026), the latest release at the time of writing.

## Archetypes in one minute

JECS is an **archetype-based** ECS. Every entity with exactly the same set of components is stored together in an *archetype*: a table of columns, one column per component, one row per entity.

That has two consequences you will feel constantly:

- **Queries are fast** because they match whole archetypes rather than checking entities one by one. A query for `Position, Velocity` just walks the archetypes that contain both columns.
- **Changing an entity's set of components moves it** to a different archetype. Adding or removing a component is therefore more expensive than updating a value that is already there.

**Archetypes form as you add components**

```luau
local e1 = world:entity()
world:set(e1, Position, Vector3.zero) -- creates archetype [Position]
world:set(e1, Velocity, Vector3.zero) -- moves e1 to archetype [Position, Velocity]

local e2 = world:entity()
world:set(e2, Position, Vector3.zero) -- archetype [Position] already exists
world:set(e2, Velocity, Vector3.zero) -- [Position, Velocity] already exists
world:set(e2, Mass, 100)              -- creates [Position, Velocity, Mass]

-- Updating a value that is already present is cheap: no move happens.
world:set(e1, Position, Vector3.new(1, 2, 3))
```

## What you get with JECS

| Feature | Why it matters |
| --- | --- |
| **Relationships** | First-class entity-to-entity links (`ChildOf`, `Owns`, `Likes`) with wildcard queries. Hierarchies and inventories without extra data structures. |
| **Cached queries** | Queries you iterate every frame can cache their matching archetypes, making iteration almost free. |
| **Hooks and signals** | Run code when a component is added, changed or removed. One hook per component, or any number of signal listeners. |
| **Cleanup traits** | Declare what happens to related entities when something is deleted (remove the link, or delete the dependents). |
| **Entity generations** | Ids carry a generation counter, so a stale handle to a deleted entity is detected instead of silently pointing at a recycled one. |
| **DEBUG mode** | `jecs.world(true)` turns on runtime checks that catch misuse early during development. |
| **Tiny API** | A world with about twenty methods. You can hold the whole thing in your head. |

**A taste of what the table promises**

```luau
local pair, ChildOf = jecs.pair, jecs.ChildOf
world:add(turret, pair(ChildOf, tank))   -- a relationship: turret is a child of tank
world:delete(tank)                       -- cleanup trait: the turret goes with it
local moving = world:query(Position, Velocity):cached() -- a cached query
```

All four are lessons of their own later. Recognize the shapes for now.

> **Where the official docs live now**
>
> Since JECS 0.10 the documentation is shipped *inside the repository* as runnable Luau files: the `how_to/` folder is a numbered tour, `examples/` shows real patterns and `modules/` contains copy-in utilities (the Jabby debugger, observers, `collect`). The old docs website is offline, so if you see tutorials referring to `jecs.World.new()` or `OnSet`, they are out of date. This course uses the current API throughout.

JECS also sits inside a healthy ecosystem: **Jabby** for debugging, **Planck** for scheduling systems, and buffer-based networking libraries such as **ByteNet**, **Zap** and **Blink** for replication. Those are covered in the later modules once you are fluent with the core.

**Key points**

- JECS is a flecs-inspired, archetype-based ECS in pure Luau (current release 0.11.0).
- Same components ⇒ same archetype. Queries match archetypes, so they are fast.
- `set` on an existing component is cheap; `add`/`remove` move entities between archetypes.
- Relationships, cached queries, hooks/signals and cleanup traits are built in.
- The official docs are the `how_to/` files in the JECS repository.
