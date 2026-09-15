# JECS reference

*JECS 0.11.0, released March 10, 2026. Checked on September 7, 2026.*

---

## World & entities

### `jecs.world`

```luau
jecs.world(debug: boolean?) -> World
```

Creates a world. Pass `true` to enable DEBUG checks on `add`, `set` and `remove`: stale entity handles, unregistered ids and wildcard misuse.

> `jecs.World.new()` still exists as a legacy alias; prefer `jecs.world()`.
>
> The checks cover `add`, `set` and `remove` only. `world:delete` is bound before the DEBUG block installs its wrapper and is never rebound, so the guard against structural changes *during* a deletion does not currently fire — do not rely on it.

### `world:entity`

```luau
world:entity() -> Entity
world:entity(id: number) -> Entity
```

Creates a new entity (recycling freed indices with an incremented generation), or creates/ensures an entity with a specific id, bypassing the entity range.

> The explicit-id form is not free of side effects: it fills the sparse array up to `id` and sets `max_id = id`. If that id is above a `world:range` ceiling, the *next* plain `world:entity()` computes `max_id + 1` and fails its range assert. Do not combine `world:range` with mirroring foreign ids — map them instead.

### `world:delete`

```luau
world:delete(entity: Entity)
```

Removes every component, runs `OnRemove` hooks with `delete = true`, applies cleanup traits, then frees the id.

### `world:contains`

```luau
world:contains(entity: Entity) -> boolean
```

True if the entity is alive: index in use and generation matches.

### `world:exists`

```luau
world:exists(entity: Entity) -> boolean
```

True if the index slot is occupied, ignoring generation. Used by replication code that maps ids.

### `world:clear`

```luau
world:clear(entity: Entity)
```

Removes all components from the entity but keeps it alive.

### `world:range`

```luau
world:range(begin: number, end: number?)
```

Restricts the ids this world allocates to a range (end is exclusive), pre-populating the sparse array up to `begin`.

> The changelog presents this as the way to receive server ids without mapping them. On 0.11.0 that combination does not work: one `world:entity(server_id)` above the ceiling breaks every later plain allocation with `Entity is outside range`. Use it to partition realms whose entities never cross, and map ids that do.

### `world:cleanup`

```luau
world:cleanup()
```

Deletes empty archetypes. Call occasionally; profile the interval.

### `jecs.ECS_ID / jecs.ECS_GENERATION`

```luau
jecs.ECS_ID(entity) -> number
jecs.ECS_GENERATION(entity) -> number
```

Decode the 24-bit index and generation packed into an entity id.

## Components & data

### `world:component`

```luau
world:component() :: jecs.Id<T>
```

Registers a component id (range 1–256) with the `jecs.Component` trait. Cast to `jecs.Id<T>` (or `jecs.Entity<T>`) to type its data.

### `jecs.component / jecs.tag / jecs.meta`

```luau
jecs.component() :: jecs.Id<T>
jecs.tag() -> Entity
jecs.meta(id, jecs.Name, "Name")
```

Preregister components, tags and metadata before `jecs.world()`; they are allocated when the world is created. Declare everything first.

> Preregistered ids are allocated when `jecs.world()` runs, so the module that declares them must be required **before** the module that creates the world. That is Project 2's first task.

### `world:set`

```luau
world:set(entity, component: Id<T>, value: T)
```

Adds the component (moving the entity to a new archetype) or updates it in place. Fires `OnAdd` on first add, `OnChange` on update. Not for tags.

### `world:get`

```luau
world:get(entity, a: Id<A>) -> A?
world:get(entity, a, b, c, d) -> (A?, B?, C?, D?)
```

Reads up to four components. Returns `nil` for missing components or dead entities.

### `world:has`

```luau
world:has(entity, ...ids) -> boolean
```

True only if the entity has all of the (up to four) ids. Works with pairs and wildcard pairs.

### `world:add`

```luau
world:add(entity, id)
```

Adds an id without data (tags, relationship pairs, traits). Idempotent.

### `world:remove`

```luau
world:remove(entity, id)
```

Removes an id. Idempotent. `OnRemove` runs before removal, and `delete` is always `nil` here — `world_remove` calls the hook with two arguments. Test it with `if delete then`, never `== false`.

### `jecs.is_tag`

```luau
jecs.is_tag(world, id) -> boolean
```

True if the id (or both halves of a pair) has no data.

### `jecs.Name`

```luau
world:set(Position, jecs.Name, "Position")
```

Built-in string component used by debuggers and replication to identify components.

### `Singletons`

```luau
world:set(TimeOfDay, TimeOfDay, 5)
world:get(TimeOfDay, TimeOfDay)
```

A world-level value is a component stored on its own id.

### `jecs.bulk_insert / jecs.bulk_remove`

```luau
jecs.bulk_insert(world, entity, { ids }, { values })
jecs.bulk_remove(world, entity, { ids })
```

Add or remove several ids in a single archetype move.

## Queries

### `world:query`

```luau
for entity, a, b in world:query(A, B) do end
```

Yields every entity that has all terms, with values in term order (typed up to eight terms). Data-less ids yield `nil`.

### `query:with / query:without`

```luau
world:query(A):with(Tag):without(Dead)
```

Require or exclude ids without yielding their values. Call before `:cached()`.

### `query:cached`

```luau
local q = world:query(A, B):cached()
```

Caches the list of matching archetypes and keeps it updated. Create once (in system setup), iterate every frame.

### `query:fini`

```luau
q:fini()
```

Unregisters a cached query so it can be garbage collected.

### `query:iter / query:archetypes`

```luau
local it = q:iter()
for _, archetype in q:archetypes() do end
```

Raw iterator function, or the matched archetypes (`entities`, `columns_map[Component]`) for hand-inlined hot loops.

### `query:has`

```luau
if q:has(entity) then end
```

True if the entity would be yielded by this query. Available on plain and cached queries; the cheapest way to test one entity against a filter set without iterating.

> Introduced in *Filters: with and without*; it respects `:with` and `:without`, which is what makes `if enemies:has(target) then` correct rather than merely cheap.

### `world:each`

```luau
for entity in world:each(id) do end
```

Iterates the entities that have a single id (no data).

### `Iteration safety`

```luau
-- archetypes are walked backwards
```

Removing or deleting the current entity during iteration is supported. Treat an uncached query object as single-pass; build a new one or cache it.

> Single-pass is not a style note: the second `for` over the same uncached query object yields **nothing**, silently. A cached query is reusable.

### `Filter calls do not accumulate`

```luau
world:query(A):with(T1, T2)  -- yes
world:query(A):with(T1):with(T2)  -- no
```

`:with()` and `:without()` each assign their filter list rather than appending, so a second call replaces the first with no error. Pass every id to one call. Mixing `:with(…):without(…)` is fine.

## Relationships

### `jecs.pair`

```luau
jecs.pair(relationship, target) -> Id
```

Encodes two entities into one id. Add, set, has, remove and query it like a component. Data type comes from the relationship if it is a component, else from the target.

### `jecs.ChildOf`

```luau
world:add(child, pair(jecs.ChildOf, parent))
```

Built-in hierarchy relationship: exclusive (one parent) with `(OnDeleteTarget, Delete)` so deleting a parent deletes its subtree, children first.

### `world:parent / world:children`

```luau
world:parent(entity) -> Entity?
for child in world:children(parent) do end
```

Shorthands for the first `ChildOf` target and for `query(pair(ChildOf, parent))`.

### `jecs.Wildcard`

```luau
pair(Likes, jecs.Wildcard)  -- any target
pair(jecs.Wildcard, alice) -- any relationship
```

Matches either half of a pair in queries and `has`. Yields an entity once. Cannot be used with add/set/remove. Alias: `jecs.w`.

### `world:target`

```luau
world:target(entity, relationship, index: number?) -> Entity?
```

Returns the nth target (0-based) of a relationship, or `nil`. Loop until `nil` to enumerate all targets.

### `jecs.IS_PAIR / pair_first / pair_second`

```luau
jecs.IS_PAIR(id) -> boolean
jecs.pair_first(world, pair) -> Entity
jecs.pair_second(world, pair) -> Entity
```

Inspect pair ids and recover their alive elements.

### `jecs.Rest`

```luau
local Previous = jecs.Rest
world:set(e, pair(Previous, Position), position)
```

A built-in, otherwise-unused entity id, handy as a marker relation when you need one and do not care what it is. The official change-tracking example uses it as the "previous value" relation; a `world:entity()` of your own works identically and reads better.

## Cleanup traits & Exclusive

### `(OnDelete, Remove)`

```luau
world:add(C, pair(jecs.OnDelete, jecs.Remove))
```

Default. Deleting the id removes it from every entity.

### `(OnDelete, Delete)`

```luau
world:add(C, pair(jecs.OnDelete, jecs.Delete))
```

Deleting the id deletes every entity that has it. Cascading; use sparingly.

### `(OnDeleteTarget, Remove)`

```luau
world:add(Rel, pair(jecs.OnDeleteTarget, jecs.Remove))
```

Deleting a target removes `(Rel, target)` from holders. Recommended for most relationships.

### `(OnDeleteTarget, Delete)`

```luau
world:add(Rel, pair(jecs.OnDeleteTarget, jecs.Delete))
```

Deleting a target deletes the holders. Strict hierarchies (what `ChildOf` uses).

### `jecs.Exclusive`

```luau
world:add(Rel, jecs.Exclusive)
```

At most one target per entity; adding another replaces it in place. Configure traits before the id is first used.

## Hooks, signals & observers

### `jecs.OnAdd`

```luau
world:set(C, jecs.OnAdd, function(entity, id, data) end)
```

Runs after the value is stored, only the first time the component is added (`data` is `nil` for `world:add`). One per component; set before first use.

### `jecs.OnChange`

```luau
world:set(C, jecs.OnChange, function(entity, id, data) end)
```

Runs when `set` updates a component the entity already has. Replaced `OnSet` in 0.6.

### `jecs.OnRemove`

```luau
world:set(C, jecs.OnRemove, function(entity, id, delete) end)
```

Runs before removal; data is still readable. `delete` is true during `world:delete`. No structural changes inside (0.11).

### `world:added / changed / removed`

```luau
local unsub = world:added(C, function(entity, id, value, oldarchetype) end)
world:changed(C, function(entity, id, value, oldarchetype) end)
world:removed(C, function(entity, id, delete) end)
```

Multi-listener signals (core since 0.9). Each returns an unsubscribe function. Do not set a raw hook on the same component afterwards.

> `added` and `changed` also receive `oldarchetype`, the archetype the entity was in before the write. Compare it against a query's archetype set to tell "just entered the query" from "changed while already matching" — that is how the bundled `OB` observer module works.
>
> The final project's `hud` system is the reason to prefer these over a raw hook: a component has one hook slot but any number of signal listeners, and `world:removed` fires during `world:delete` too.

### `jecs.ArchetypeCreate / ArchetypeDelete`

```luau
world:added(jecs.ArchetypeCreate, fn)
```

Built-in ids that fire when an archetype containing a given component is created or destroyed. This is how cached queries keep their archetype lists current; the `OB` module uses them too.

### `OB.observer / OB.monitor`

```luau
OB.observer(world:query(A, B), function(entity) end)
local m = OB.monitor(world:query(A, B)); m.added(fn); m.removed(fn); m.disconnect()
```

Copy-in module from the JECS repo (`modules/OB/module.luau`). Reacts when an entity starts/stops matching a whole query.

## Luau types

### `jecs.World`

```luau
local function system(world: jecs.World) end
```

The world type.

### `jecs.Entity<T> / jecs.Id<T>`

```luau
local Position = world:component() :: jecs.Id<Vector3>
local e: jecs.Entity = world:entity()
```

Phantom-typed ids. `jecs.Entity` with no argument is a plain entity/tag.

### `jecs.Pair<First, Second>`

```luau
jecs.Pair<Owns, Car>
```

Type of a pair id; resolves to `Second`'s data when `First` is a tag (needs the new Luau type solver).

## Ecosystem APIs

Third-party packages, not part of jecs. Versions and install strings are in the ecosystem table.

### `jabby.set_check_function`

```luau
jabby.set_check_function(function(player) return player.UserId == me end)
```

Decides who may open the inspector. Call on every VM, before `register`.

> Without it the inspector is open to everyone on the server.

### `jabby.register`

```luau
jabby.register({ name = "server", applet = jabby.applets.world, configuration = { world = world } })
```

Publishes a world or scheduler to the home app. Applets: `jabby.applets.world`, `jabby.applets.scheduler`.

> The world applet reads `jecs.Name` off component ids, so naming components is what makes the UI readable. `configuration` also takes `get_entity_from_part` and an `entities` table.

### `jabby.scheduler.create`

```luau
local s = jabby.scheduler.create()
```

Creates a Jabby scheduler that times and pauses systems.

> Independent of Planck; the scheduler applet shows one row per registered system.

### `scheduler:register_system / scheduler:run`

```luau
local id = s:register_system({ name = "movement", phase = "Heartbeat" })
s:run(id, run_movement, dt)
```

Registers one id per system, then wraps the call: records the frame time and skips it while paused.

> Extra arguments to `run` are forwarded to the system.

### `jabby.obtain_client`

```luau
local client = jabby.obtain_client()
client.spawn_app(client.apps.home, nil)
```

Client entry point; opens the home app, usually bound to a key with `ContextActionService`.

> Lists every registered world and scheduler on every VM you are allowed to see.

### `Planck.Scheduler.new`

```luau
Planck.Scheduler.new(world, state):addPlugin(PlanckRunService.Plugin.new())
```

Creates a scheduler. Every argument after the first is passed to every system, so a system is `(world, state)`.

> Without a plugin the scheduler has no clock.

### `scheduler:addSystem / addSystems`

```luau
scheduler:addSystem(render)
scheduler:addSystem({ name, system, phase, runConditions })
scheduler:addSystems({ a, b }, Phases.PostSimulation)
```

Adds one system or several. With no phase, the RunService plugin's default is `Phases.Update`.

> `removeSystem` and `replaceSystem` exist too. Systems inside a phase run in insertion order. Custom phases are placed with `scheduler:insert(phase, RunService, "Heartbeat")` or `scheduler:insertAfter(phase, Phases.Update)`.

### `PlanckRunService.Phases`

```luau
Phases.First → PreUpdate → Update → PostUpdate → Last (Heartbeat)
Phases.PreRender / PreAnimation / PreSimulation / PostSimulation
Planck.Phase.Startup
```

The phases a system can be placed in, in the order they run each frame.

> `Heartbeat` is the legacy name for `PostSimulation`, so those two pipelines ride the same engine event; order between them is connection order.

### `Planck.timePassed / runOnce / onEvent`

```luau
runConditions = { Planck.timePassed(1 / 20) }
```

Run conditions: the scheduler skips the system on frames where the condition answers no.

> **Run conditions are not dependencies.** They gate the one system they are attached to and say nothing about order.

### `replecs.create_server / create_client`

```luau
local server = replecs.create_server(world)
local client = replecs.create_client(world)
```

The two ends of replecs replication. Markers: `replecs.shared`, `replecs.networked`, `replecs.reliable`, `replecs.unreliable`, `replecs.relation`.

> `require(replecs)` must run **before** `jecs.world()`: its markers are preregistered ids. There is no `replecs.pair`, whatever the networking guide shows.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [jecs](https://github.com/Ukendio/jecs) | ECS | 0.11.0 | `ukendio/jecs@0.11.0 · @rbxts/jecs` |
| [Jabby](https://github.com/alicesaidhi/jabby) | Debugger / inspector | 0.6.1 | `alicesaidhi/jabby@0.6.1` |
| [Planck](https://github.com/YetAnotherClown/Planck) | Scheduler (phases, pipelines) | 0.2.0 (0.3.0-rc.2) | `yetanotherclown/planck@0.2.0 + planck-runservice, planck-jabby` |
| [replecs](https://github.com/PepeElToro41/replecs) | Replication | 0.3.1 (0.4.0-rc.3 for jecs 0.11) | `pepeeltoro41/replecs` |
| [Zap](https://github.com/red-blox/zap) | Networking IDL → buffers | 0.6.29 | `rokit add red-blox/zap` |
| [Blink](https://github.com/1Axen/blink) | Networking IDL → buffers | 0.18.8 | `rokit add 1Axen/blink` |
| [ByteNet](https://github.com/ffrostfall/ByteNet) | Networking (runtime packets) | 0.4.6 | `ffrostflame/bytenet@0.4.6` |
| [jecs-utils / jecs-vide](https://github.com/PepeElToro41/jecs-utils) | Query helpers · Vide bindings | 1.4.4 / 1.0.8 | `pepeeltoro41/jecs-utils, pepeeltoro41/jecs-vide` |
| [rubine](https://github.com/Mark-Marks/rubine) | Scheduler (alternative) | 0.3.0-alpha.2 | `mark-marks/rubine` |
| [Wally](https://github.com/UpliftGames/wally) | Package manager — the install path Installing JECS teaches | 0.3.2 (5 June 2023) | `rokit add UpliftGames/wally` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Filesystem to Roblox sync | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Editor tooling | 1.69.0 (18 July 2026) | `VS Code extension JohnnyMorganz.luau-lsp` |
| [Matter](https://github.com/matter-ecs/matter) | The ECS this course maps from | 0.8.5 (9 December 2024) | `matter-ecs/matter` |
| [jecs-template](https://github.com/LocalPulse/jecs-template) | Starter repository | December 2025 snapshot | `clone LocalPulse/jecs-template` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- The old docs website (ukendio.github.io/jecs) is offline; the official docs are the `how_to/` files inside the JECS repository.
- `jecs.World.new()` still works but `jecs.world()` is the documented constructor (0.7+).
- `OnSet` was replaced by `OnChange` in 0.6; `OnAdd` now receives the value and runs after it is stored.
- `Query:drain`, `Query:next` and `Query:replace` were removed in 0.4.
- The in-repo CHANGELOG stops at 0.8.0; newer release notes live on the GitHub Releases page.
- There is no official jecs package on pesde; `marked/jecs` and `tazmondo/jecs` are community mirrors.
- Jabby's install docs still say 0.2.0; use 0.6.1 with jecs 0.11.
- replecs' networking guide is written against a `replecs.pair` marker. The module exports `replecs.relation` (plus a `Relation` alias) and has no `pair` field, so the documented code errors with a `nil` relationship.
- The changelog says `world:range` "makes it possible to receive ids from a server that don't have to be mapped to local ids". On 0.11.0 it does not: `world:entity(id)` moves the allocator's `max_id`, so the next plain `world:entity()` fails its range assert. Map ids the way `examples/networking/networking_recv.luau` does.
- `how_to/110_hooks.luau` still says "You can call world:add, world:remove, and world:set inside OnRemove hooks. That's fine." Since 0.11 structural changes in `OnRemove` are unsupported; some paths hard-error and the rest merely happen to work.
- `jecs.world(true)` is often described as catching structural changes during deletion. It does not: `world.delete` is bound before the DEBUG block and never rebound, so only `add`, `set` and `remove` are checked.
- `world:targets(entity, relationship)` exists on `main` but is not in 0.11.0; loop `world:target` with an increasing index instead.
- `sapphire` and `sapphire-jecs` were archived in June 2025 and still appear in scheduler tutorials. Use Planck, or rubine if you accept alpha software.
- Planck's documentation, including its JECS guide, pins jecs `0.5.3` — before `jecs.world()`, `OnChange` and the core signals. Take the concepts from the docs and the versions from your lockfile.
- Zap examples written before 0.6.18 present callbacks as the only call mode. Polling has existed since then and is the mode an ECS wants.
- ByteNet's README reads as current, but the project has seen little activity since 2025. The author is `ffrostfall` on GitHub and `ffrostflame` on the Wally registry — the mismatch is not a typo in your manifest.
- The JECS repo's `modules/` folder (`OB`, `collect`, `testkit`, `lifetime_tracker`, `entity_visualizer`, a vendored Jabby) is plain `.luau` to copy into your project, not a package to install. Guides that say `wally install` for `OB` are wrong.
- Wally has shipped one release, 0.3.2, since 5 June 2023. Writing `0.3.x` in a toolchain file is not sloppiness and an installer that pulls `0.3.2` is not out of date — there is nothing newer to pull.
