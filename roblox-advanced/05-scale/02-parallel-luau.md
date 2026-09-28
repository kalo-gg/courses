# Parallel Luau

*Put scripts under Actors, read the world in the parallel phase, and change it only after task.synchronize.*

Roblox Advanced · Scale · 12 min

---

A server runs 500 NPCs, and every frame one script measures how far each NPC is from the nearest player. It is plain arithmetic, yet it all runs on one core while the others sit idle, and the frame gets longer with every NPC you add.

**Parallel Luau** lets scripts run on several cores at once, under one rule: in the parallel phase a script may read the world, but it must return to the serial phase to change it.

## Actors and phases

Only a script under an **Actor**, a kind of Model that marks one unit of work, can run in parallel. Scripts under the same Actor still run one after another, so parallel work needs several Actors, split by logic: one per NPC or per squad. Each Actor runs its own Luau VM, so a ModuleScript required under two Actors runs twice and shares no state.

A script starts serial. `task.desynchronize()` resumes it in the next parallel phase, and `task.synchronize()` in the next serial one. `ConnectParallel` starts a handler in the parallel phase.

**A Script inside each NPC, whose model is an Actor**

```luau
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local npc = script:GetActor()
local root = npc.HumanoidRootPart
local humanoid = npc.Humanoid

RunService.Heartbeat:ConnectParallel(function()
	-- Parallel phase: read the world, write only locals
	local best, bestDistance = nil, 80
	for _, player in Players:GetPlayers() do
		local character = player.Character
		local target = character and character:FindFirstChild("HumanoidRootPart")
		if target then
			local distance = (target.Position - root.Position).Magnitude
			if distance < bestDistance then
				best, bestDistance = target, distance
			end
		end
	end
	if best == nil then
		return
	end

	task.synchronize()
	-- Serial phase: MoveTo changes the world
	humanoid:MoveTo(best.Position)
end)
```

Everything above `task.synchronize()` only reads, so hundreds of these handlers can share the cores. `MoveTo` is Unsafe, so it waits for the serial phase.

## Thread safety

Every API member has a thread-safety level, shown on its reference page, and a member with no level counts as Unsafe. In the parallel phase the engine detects unsafe access and blocks it.

| Level | Properties | Functions |
| --- | --- | --- |
| Unsafe | Cannot be read or written in parallel | Cannot be called in parallel |
| Read Parallel | Can be read, not written | — |
| Local Safe | Usable within the same Actor; other Actors may only read | Callable within the same Actor, not by other Actors in parallel |
| Safe | Can be read and written | Can be called |

Most properties are Read Parallel, and most methods and events are Unsafe. Hence the usual shape: read in parallel, write in serial.

**Switching phase mid-handler**

```luau
-- A Script inside each player's Actor, validating their shots
remote.OnServerEvent:Connect(function(player: Player, aim: Vector3)
	-- Serial: the set-up, as in the guide's example
	local params = RaycastParams.new()
	params.FilterDescendantsInstances = { player.Character }

	task.desynchronize() -- parallel: Raycast is Safe
	local origin = player.Character.HumanoidRootPart.Position
	local result = workspace:Raycast(origin, (aim - origin).Unit * 300, params)

	task.synchronize() -- serial: now it may change the world
	if result and result.Instance.Name == "Target" then
		result.Instance:Destroy()
	end
end)
```

`task.desynchronize()` moves a serial handler into the parallel phase partway through. The Parallel Luau guide's hit-validation example has this shape. `ConnectParallel` is the more efficient choice when the whole handler can start in parallel.

> **Before you go parallel**
>
> `require` cannot run in the parallel phase, so require modules at the top of the script. `task.desynchronize` and `task.synchronize` raise an error in a script that is not under an Actor.

> **In the sandbox**
>
> The sandbox runs one thread, so it shows the rules but not the speed-up: measure that in Studio with the tools in the Profiling lesson. Your file runs under no Actor, so `sim.runInActor(actor, fn, ...)` runs `fn` as if it were a Script inside `actor`.

**Key points**

- Only scripts under an Actor can run in parallel, and scripts under one Actor run one after another: use several Actors, split by logic.
- `ConnectParallel` and `task.desynchronize()` enter the parallel phase; `task.synchronize()` returns to the serial one.
- In parallel, read properties, call Safe members and write locals. Change instances only after `task.synchronize()`.
- A member with no thread-safety level is Unsafe. Require modules before going parallel.
