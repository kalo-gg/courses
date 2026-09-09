# Scheduling with Planck

*Phases, pipelines and run conditions for running JECS systems with the Planck scheduler.*

JECS · The JECS Ecosystem · 12 min

---

Systems and the frame ran systems from one `RunService.Heartbeat` connection and an explicit list. As a project grows you want named phases, run conditions and a place for tooling to hook into. JECS ships no scheduler; the community answer is **Planck** by YetAnotherClown, an ECS-agnostic scheduler inspired by Bevy and flecs. Systems are plain functions and the world is just an argument, so no JECS plugin is needed.

**wally.toml**

```toml
[dependencies]
jecs = "ukendio/jecs@0.11.0"
Planck = "yetanotherclown/planck@0.2.0"
PlanckRunService = "yetanotherclown/planck-runservice@0.2.0"
```

`0.2.0` (July 2025) is the latest stable release of both packages. A `0.3.0-rc.2` pre-release (May 2026) exists if you want to track the next line. `yetanotherclown/planck-jabby@0.2.0` is the optional bridge that registers your systems with Jabby's scheduler applet.

**Creating a scheduler — shared/scheduler.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Planck = require(ReplicatedStorage.Packages.Planck)
local PlanckRunService = require(ReplicatedStorage.Packages.PlanckRunService)
local world = require(ReplicatedStorage.Shared.world)

local state = {} -- anything you want every system to receive

local scheduler = Planck.Scheduler.new(world, state)
	:addPlugin(PlanckRunService.Plugin.new())

return scheduler
```

Every argument after the first to `Scheduler.new` is passed to every system, so a system's signature is `(world, state)`. The RunService plugin is what connects phases to the Roblox frame; without a plugin the scheduler has no clock. The factory shape from Systems and the frame still works: build cached queries once and register the returned function.

> **The docs pin old versions**
>
> Planck's documentation, including its JECS guide, still pins JECS `0.5.3`, which is several major versions behind (no `jecs.world()`, no signals). Take the *concepts* from the docs, take *versions* from the registry and your lockfile, and treat the API in this lesson (checked against the 0.2 source) as the reference. When something differs, read the source; it is small.

## Pipelines and phases

| Name | Runs on | Notes |
| --- | --- | --- |
| `Phases.PreRender` | `RunService.PreRender` | Camera and rendering work. |
| `Phases.PreAnimation` | `RunService.PreAnimation` |  |
| `Phases.PreSimulation` | `RunService.PreSimulation` | Before physics. |
| `Phases.PostSimulation` | `RunService.PostSimulation` | After physics. |
| `Phases.First` → `PreUpdate` → `Update` → `PostUpdate` → `Last` | `RunService.Heartbeat` | Five ordered phases inside the Heartbeat pipeline. **`Update` is the default.** Note `Heartbeat` is the legacy *name* for `PostSimulation`, so this pipeline and the one above ride the same engine event. |
| `Planck.Phase.Startup` | Once, when the scheduler starts | Flanked by `PreStartup` and `PostStartup`. |

Listed in the order they run each frame. `PlanckRunService.Phases` exposes all of these; a `Pipelines` table groups the RunService-level ones. Systems inside a phase run in the order they were added.

A **phase** is a named slot inside a **pipeline**, and a pipeline is one engine event. Everything under `Heartbeat` runs in the listed order every frame, `PreRender` only exists on the client, and `Startup` runs once. Placing a system is two decisions — which event it rides, and where among its neighbors — and the second replaces the hand-written list.

**Adding systems**

```luau
local Phases = PlanckRunService.Phases

-- A function with no phase goes to Phases.Update
scheduler:addSystem(render)

-- Startup phases run exactly once
scheduler:addSystem(spawnMap, Planck.Phase.Startup)

-- A system table gives you a name (shown in Jabby) and run conditions
scheduler:addSystem({
	name = "move",
	system = move,
	phase = Phases.Update,
	runConditions = { Planck.timePassed(1 / 30) },
})

-- Several systems at once, in insertion order, all in one phase
scheduler:addSystems({ gravity, collisions, damage }, Phases.PostSimulation)
```

`scheduler:removeSystem(damage)` and `scheduler:replaceSystem(move, moveV2)` exist too, for systems you swap at runtime.

## Run conditions

A **run condition** decides, each time its phase runs, whether the system should run. Planck ships four:

- `Planck.timePassed(seconds)`: at most once per interval.
- `Planck.runOnce()`: the first time only.
- `Planck.onEvent(instance, "EventName")`: true when the event fired since the last run.
- `Planck.isNot(condition)`: inverts another condition.

**Run conditions are not dependencies.** A condition only *skips* a system; order comes from phases and insertion order, so every system must be correct even when others were skipped.

```luau
local playerJoined, collectJoins = Planck.onEvent(Players, "PlayerAdded")

local function spawnPlayers()
	-- collectJoins() returns an iterator, not a table
	for i, player in collectJoins() do
		print(i, player.Name)
	end
end

scheduler:addSystem({
	name = "spawn_players",
	system = spawnPlayers,
	runConditions = { playerJoined },
})

scheduler:addRunCondition(replicate, Planck.timePassed(1 / 20))
scheduler:addRunCondition(printBanner, Planck.runOnce())
```

`onEvent` is Planck's answer to the `collect` helper: events are queued and drained inside a system, at a predictable point in the frame. Its full signature is `hasNewEvent, collectEvents, getDisconnectFn = onEvent(...)`.

**Key points**

- Planck is an ECS-agnostic scheduler; JECS needs no plugin. Install `planck@0.2.0` + `planck-runservice@0.2.0` and ignore the docs' stale JECS pin.
- `Planck.Scheduler.new(world, ...)` passes its extra arguments to every system; `addPlugin(PlanckRunService.Plugin.new())` binds phases to the frame.
- Heartbeat has `First`, `PreUpdate`, `Update` (default), `PostUpdate`, `Last`; `Planck.Phase.Startup` runs once.
- `timePassed`, `runOnce`, `onEvent` and `isNot` are run conditions, and run conditions are not dependencies.
- Custom phases exist (`insert` / `insertAfter`); keep the file layout from Roblox Integration — Planck's docs recommend the same one.
