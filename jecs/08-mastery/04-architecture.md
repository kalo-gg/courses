# Planning the wave-survival server

*Lay out the final project file by file: what goes where, which system runs before which, and how systems talk.*

JECS · Best Practices & Capstone · 10 min

---

The final project assembles a wave-survival server from the two projects you already built and four new systems. Before writing any of them, decide three things: **where each file lives**, **the order systems run**, and **which data each system reads and writes**. The third decides the second, and the second is a single list in the startup script.

**The plan**

```luau
-- components (shared):  Position, Velocity, Health, IncomingDamage, Bounty, Model,
--                       Turret, Reload, Enemy, Dead, Score, Wave, Targeting
-- systems, in order:    hud -> spawn -> spawn_models -> movement -> render
--                       -> turrets -> damage -> reap
-- hud:       signals keep alive/kill counters (subscribes first so it sees every enemy)
-- spawn:     every wave, create N enemies as children of a wave entity   (Project 1)
-- movement:  Position += Velocity * dt                                  (Project 1)
-- render:    copy Position into each Model                              (Project 2)
-- turrets:   write IncomingDamage on enemies in range, remember the target
-- damage:    apply IncomingDamage, tag Dead
-- reap:      bank Bounty into Score, delete the Dead (Model hooks destroy Instances)
```

## What goes where

| What | Where | When |
| --- | --- | --- |
| Component and tag ids, `jecs.Name`s | `Shared/components.luau` | Module load, before the world |
| `jecs.world()` | `Shared/world.luau` | First require, after `components` |
| Hooks and cleanup traits | startup script | Once, before any system runs |
| Server-only systems (players, turrets, replication send) | `Server/systems/` | `Heartbeat` |
| Client-only systems (input, render, camera) | `Client/systems/` | `Heartbeat`, `PreRender` for camera work |
| Simulation both realms run | `Shared/systems/` | `Heartbeat` |

The layout from Roblox Integration; Planck's docs recommend the same one, so adopting a scheduler changes nothing here.

Two rules you already know push every project into this shape: *declare ids before the world* and *configure hooks and traits before first use*. Add a third: each realm's startup script is the **only** place that knows the order. Name systems after what they do, in `snake_case`, and keep the file name, the factory name and the entry in the list the same word, so a stack trace, a Jabby timing row and a `grep` all lead to the same place.

**startup.server.luau, in outline**

```luau
local world = require(ReplicatedStorage.Shared.world)
local ct = require(ReplicatedStorage.Shared.components)

-- Traits, once, before anything uses Targeting
world:add(ct.Targeting, jecs.Exclusive)
world:add(ct.Targeting, jecs.pair(jecs.OnDeleteTarget, jecs.Remove))

-- Hooks: OnAdd stamps entityId on the Model, OnRemove destroys it
-- (both written in Project 2)

local factories = {
	require(script.Parent.systems.players)(world, Players, collect),
	require(ReplicatedStorage.Shared.systems.spawn)(world),
	require(ReplicatedStorage.Shared.systems.movement)(world),
	require(script.Parent.systems.render)(world, CFrame),
}

RunService.Heartbeat:Connect(function(dt)
	for _, system in factories do
		system(dt)
	end
end)
```

Dependencies go in as arguments, so the system files have no Roblox globals and run under `luau` and in this sandbox.

## Systems talk through data

| System | Reads | Writes |
| --- | --- | --- |
| spawn | `Wave`, `SpawnPoint`, `BasePosition` | new enemies: `Position`, `Velocity`, `Health`, `Bounty`, `Enemy`, `ChildOf` |
| movement | `Position`, `Velocity` | `Position` |
| turrets | `Position`, `Turret`, `Reload`, enemies' `Position` | `IncomingDamage`, `Reload`, `pair(Targeting, enemy)` |
| damage | `Health`, `IncomingDamage` | `Health`, removes `IncomingDamage`, adds `Dead` |
| reap | `Bounty` with `Dead` | `Score`; deletes the entity |
| hud | signals on `Enemy`, `Dead` | counters outside the world |

Read down the *Writes* column: a system must run after everything that writes what it reads. That is the whole ordering argument.

One more rule the table does not show: a system never calls another system. `turrets` does not tell `damage` to subtract health, it writes `IncomingDamage` and lets `damage` find it next in the list. That is what makes the order list the only place order lives, and what lets you delete, reorder or test any one of these files without opening the others.

**Key points**

- Ids in `components.luau`, the world in `world.luau`, hooks and traits and the system list in the startup script.
- Order systems so every reader runs after its writer; write the reads/writes table before the code.
- Systems communicate through data (`IncomingDamage`, `Dead`), not direct calls.
- Inject `Players`, `collect` and `CFrame`; system files with no globals run anywhere Luau runs.
