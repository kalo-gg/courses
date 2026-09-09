# Debugging with Jabby

*Install the Jabby inspector, register your world and systems, query entities in-game, and know the fallback.*

JECS · The JECS Ecosystem · 11 min

---

JECS ships a core and stops there: no inspector, no scheduler, no replication, no migration path. That is deliberate, but it means the second half of a real project is spent choosing packages. This module installs the four that matter, starting with the one you reach for first.

Once a world has a few thousand entities, `print` stops being debugging. **Jabby**, by alicesaidhi, is the window you need: an in-game UI with an entity browser, a query box and per-system timings. The author calls it experimental, so pin your versions.

**wally.toml**

```toml
[dependencies]
jecs = "ukendio/jecs@0.11.0"
jabby = "alicesaidhi/jabby@0.6.1"
```

Jabby 0.6.x requires JECS 0.11.x. A vendored copy also ships inside the JECS repository under `modules/Jabby` if you prefer to avoid the extra dependency.

## Setup on every VM

Jabby runs a small server on each VM and talks between them. Two calls are needed on **both** realms: a check function deciding who may use it, and a registration of your world with the *world applet*.

**Shared/debug.luau (required from both startup scripts)**

```luau
local jabby = require(ReplicatedStorage.Packages.jabby)
local world = require(ReplicatedStorage.Shared.world)

-- Who may open Jabby, pause systems and edit entities. Required on every VM.
jabby.set_check_function(function(player: Player)
	return RunService:IsStudio()
end)

jabby.register({
	name = if RunService:IsServer() then "server world" else "client world",
	applet = jabby.applets.world,
	configuration = {
		world = world,
	},
})
```

The check function receives the requesting `Player`; gating on `RunService:IsStudio()` is the simplest safe default. Name components with `jecs.meta(C, jecs.Name, "C")` — the world applet reads it, and unnamed components show as bare numbers.

## Timing, pausing and querying

**startup.server.luau (excerpt)**

```luau
local scheduler = jabby.scheduler.create()

jabby.register({
	name = "server scheduler",
	applet = jabby.applets.scheduler,
	configuration = { scheduler = scheduler },
})

local run_movement = require(ReplicatedStorage.Shared.systems.movement)(world)
local movement_id = scheduler:register_system({ name = "movement", phase = "Heartbeat" })

RunService.Heartbeat:Connect(function(dt)
	scheduler:run(movement_id, run_movement, dt)
end)
```

`scheduler:run` wraps the call: it records the time taken and skips it while paused. Any extra arguments are passed to the system, and the scheduler applet shows a live table of frame times with a pause toggle for each.

On the client, `jabby.obtain_client()` returns a client whose `spawn_app(client.apps.home, nil)` opens the home app; bind it to a key such as F4. The world applet's `configuration` also takes `get_entity_from_part`, which maps a clicked `BasePart` to an entity — the `entityId` attribute pays off again — and the query box speaks the same language as `world:query`.

**The zero-dependency alternative.** `jecs.world(true)` turns on **DEBUG mode** (Worlds and entities), which errors loudly on stale handles, unregistered ids and wildcard pairs passed to `add`/`set`/`remove`. Enable it only in Studio. One caveat: it wraps `add`, `set` and `remove` and **nothing else**, so it will not catch a structural change made inside an `OnRemove` hook during deletion.

```luau
local world = jecs.world(RunService:IsStudio())

local e = world:entity()
world:delete(e)
world:set(e, ct.Position, Vector3.zero) -- DEBUG: errors here instead of corrupting a recycled entity
```

DEBUG mode is a tripwire, not an inspector: it tells you *that* something went wrong at the earliest possible moment. Jabby tells you *what the world looks like* right now. Use both.

> **Experimental: pin and gate**
>
> Keep `jabby` and `jecs` pinned to compatible versions (0.6.x with 0.11.x). No release of `planck-jabby` currently pairs with jabby 0.6.1: `0.2.0` depends on `alicesaidhi/jabby@0.3.0` and even `0.3.0-rc.2` depends on `0.5.1`, so adding either alongside `jabby@0.6.1` installs two copies of the debugger. And never ship the F4 binding without a restrictive check function: whoever passes it can pause your systems and edit entities.

**Key points**

- Jabby (`alicesaidhi/jabby@0.6.1`, needs JECS 0.11.x) is an experimental in-game inspector for JECS.
- On every VM: `jabby.set_check_function` to gate access, then `jabby.register` the world applet; name components with `jecs.Name`.
- The scheduler applet plus `scheduler:register_system` / `scheduler:run` gives per-system timings and pausing.
- Client: `jabby.obtain_client()` and `client.spawn_app(client.apps.home, nil)` on a key such as F4; the query box speaks `Position, !Dead`, `$id`, `(ChildOf, $624)`, `(OnCooldown, *)`.
- `jecs.world(true)` DEBUG mode is the zero-dependency tripwire; enable it in Studio only.
