# One loop, many parts

*Drive every tagged part from one connection over a table, and hand continuous motion to the engine.*

Roblox Fundamentals · Shipping a place · 10 min

---

The Blink script from the Events module gave each tagged part its own loop. With twelve parts nobody notices. Then a builder tags 2,000 lamps, and the server runs 2,000 loops: 2,000 threads or connections, each with its own closure, each one something to find and stop when its tag goes.

The per-part work is not the problem. The per-part *machinery* is.

## One connection, a table of parts

**One connection per turret**

```luau
local connections = {} -- [turret] = its Heartbeat connection

local function track(turret)
	connections[turret] = RunService.Heartbeat:Connect(function()
		local target = nearestRoot(turret.Position)
		if target then
			turret.CFrame = CFrame.lookAt(turret.Position, target.Position)
		end
	end)
end

local function untrack(turret)
	connections[turret]:Disconnect()
	connections[turret] = nil
end
```

`nearestRoot(position)` returns the closest HumanoidRootPart, or `nil`. 2,000 turrets means 2,000 connections, each called every frame.

**One connection for every turret**

```luau
local turrets = {} -- a set: [turret] = true

RunService.Heartbeat:Connect(function()
	for turret in turrets do
		local target = nearestRoot(turret.Position)
		if target then
			turret.CFrame = CFrame.lookAt(turret.Position, target.Position)
		end
	end
end)

local function track(turret)
	turrets[turret] = true
end

local function untrack(turret)
	turrets[turret] = nil
end

for _, turret in CollectionService:GetTagged("Turret") do
	track(turret)
end
CollectionService:GetInstanceAddedSignal("Turret"):Connect(track)
CollectionService:GetInstanceRemovedSignal("Turret"):Connect(untrack)
```

Three connections in all, however many turrets there are. Adding a turret is a table write, and removing one sets it to `nil`.

At 60 frames a second, each frame has about 16.7 ms for everything: physics, the task scheduler and every handler. Work that does not fit makes frames late, and everything waiting on a frame waits longer.

Measure before you rewrite. The Developer Console's **Script Profiler** records how much CPU time each script takes, so you know which loop is the expensive one.

## Or no loop at all

The cheapest loop is one the engine runs. A `HingeConstraint` motor spins a part and a `PrismaticConstraint` servo slides one; the physics engine solves both, with no Luau per frame. A tween moves a property from A to B without a line of yours per frame. Keep your loop for per-frame decisions only your code can make, like picking a target.

**Key points**

- A connection per part is a handler call per part per frame, plus a connection to track and disconnect for each.
- One Heartbeat over a table does the same work in one call; adding or removing a part is a table write.
- Keep per-part state as the table's value, such as the seconds since the last toggle.
- Measure with the Script Profiler before you optimise.
- Continuous motion belongs to constraints and one-off moves to tweens; loop only for what the engine cannot decide.
