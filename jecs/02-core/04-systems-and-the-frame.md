# Systems and the frame

*Writing systems as functions and running them from RunService every frame.*

JECS · Entities, Components & Systems · 12 min

---

JECS deliberately ships **no scheduler**. A system is any function that reads and writes the world. How and when it runs is up to you: a `RunService.Heartbeat` connection is enough to start, and Scheduling with Planck introduces it for larger projects.

**systems/movement.luau**

```luau
local ct = require(ReplicatedStorage.Shared.components)

return function(world: jecs.World, dt: number)
	for entity, position, velocity in world:query(ct.Position, ct.Velocity) do
		world:set(entity, ct.Position, position + velocity * dt)
	end
end
```

`world:query(A, B)` yields every entity that has both components, plus their values, in the order you listed. `ct` (component table) is the alias the JECS examples use for the shared components module. Queries & Archetypes covers queries in depth.

**server/main.server.luau**

```luau
local RunService = game:GetService("RunService")
local world = require(ReplicatedStorage.Shared.world)
local movement = require(script.Parent.systems.movement)
local gravity = require(script.Parent.systems.gravity)

local systems = { gravity, movement } -- order matters: gravity first, then movement

RunService.Heartbeat:Connect(function(dt)
	for _, system in systems do
		system(world, dt)
	end
end)
```

This tiny loop already has the two properties a scheduler needs: a **fixed order** (listed explicitly) and a **shared `dt`**. Everything else, like phases, run conditions and profiling, is convenience you can add later.

## The system-factory shape

The official JECS examples use a slightly different shape: a function that receives the world once, sets up anything it needs, and returns the per-frame function. That keeps per-system state local and avoids re-creating queries every frame.

The setup here calls `:cached()` on a query. Caching and archetypes covers it properly; for now: a cached query remembers which archetypes match, so it costs a little to build and almost nothing to iterate.

**Factory shape (used throughout the JECS examples)**

```luau
local function movement(world: jecs.World)
	-- setup runs once
	local moving = world:query(ct.Position, ct.Velocity):cached()

	-- this is the system body, run every frame
	return function(dt: number)
		for entity, position, velocity in moving do
			world:set(entity, ct.Position, position + velocity * dt)
		end
	end
end

local run_movement = movement(world)
RunService.Heartbeat:Connect(run_movement)
```

## Reading from Roblox inside systems

Systems should be *pull*-based: they run at a predictable point in the frame and read whatever has happened since. Roblox, however, is *push*-based, full of signals like `PlayerAdded` and `Touched`. The JECS repository ships a tiny helper called `collect` that turns any signal into an iterator you can drain inside a system. The sandbox on this site ships the same helper, so `require(ReplicatedStorage.Shared.collect)` works in every exercise here.

**systems/players.luau**

```luau
local Players = game:GetService("Players")
local collect = require(ReplicatedStorage.Shared.collect)

local function players(world: jecs.World)
	local players_added = collect(Players.PlayerAdded)
	local players_removing = collect(Players.PlayerRemoving)
	local by_player: { [Player]: jecs.Entity } = {}

	return function()
		for player in players_added do
			local e = world:entity()
			world:set(e, ct.Player, player)
			by_player[player] = e
		end

		for player in players_removing do
			local e = by_player[player]
			if e then
				world:delete(e)
				by_player[player] = nil
			end
		end
	end
end
```

Events are queued as they happen and processed in one place, in system order, once per frame. No callbacks mutate the world at surprising times.

> **Small systems, explicit order**
>
> Keep systems **small and single-purpose**, give each one an obvious name, and list their run order explicitly in one place. When something breaks, you will know exactly which function touched which components and when.

**Key points**

- A system is just a function over the world; JECS has no built-in scheduler.
- Run systems in an explicit order from `RunService.Heartbeat` (or a scheduler such as Planck).
- Use the factory shape to set up cached queries once and return the per-frame body.
- Drain Roblox signals inside systems with `collect` instead of mutating the world from callbacks.
