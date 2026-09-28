# Client performance

*Keep PreRender work small, keep busy interface apart from the rest, and reuse effects from a pool.*

Roblox Intermediate · Shipping multiplayer · 10 min

---

A minigun fires 20 shots a second. Each shot creates a Part for its tracer, parents it, and hands it to Debris. On a laptop nobody notices. On a low-end phone the game stutters whenever anyone holds the trigger, on every client that can see them.

Client performance is frame time: everything a client does between two frames, your code included.

## Keep PreRender small

```luau
-- Every frame, every enemy, even when nothing changed
RunService.PreRender:Connect(function()
	for _, enemy in enemies do
		enemy.Bar.Size = UDim2.fromScale(enemy.Humanoid.Health / enemy.Humanoid.MaxHealth, 1)
	end
end)

-- Each bar changes when its health does
for _, enemy in enemies do
	enemy.Humanoid.HealthChanged:Connect(function(health)
		enemy.Bar.Size = UDim2.fromScale(health / enemy.Humanoid.MaxHealth, 1)
	end)
end
```

The engine cannot start rendering a frame until `PreRender` handlers finish, so the docs say to use it sparingly. Keep it for what truly moves every frame, like a custom camera, and drive the rest from events.

Interfaces cost frame time too. A ScreenGui caches its appearance and recomputes it on the next frame after any change inside it: a descendant added or removed, or a property changed.

So a timer that ticks inside the ScreenGui holding the whole inventory makes the whole inventory recompute. Give fast-changing UI its own ScreenGui, and create elements for what is on screen, not for all 500 items.

## Pool what you make often

```luau
local free = {} -- flashes not in use, hidden

local function acquire(): Part
	local flash = table.remove(free)
	if flash == nil then
		flash = Instance.new("Part")
		flash.Shape = Enum.PartType.Ball
		flash.Size = Vector3.new(1, 1, 1)
		flash.Anchored = true
		flash.CanCollide = false
		flash.CanQuery = false
		flash.Parent = effects
	end
	return flash
end

local function release(flash: Part)
	flash.Transparency = 1 -- hidden, not destroyed
	table.insert(free, flash)
end

-- On every impact
local flash = acquire()
flash.Position = hitPosition
flash.Transparency = 0
task.delay(0.1, release, flash)
```

The first impacts create flashes. After that, a flash that has finished is reused, so the pool only grows to the most flashes ever visible at once.

A pool creates each instance once and then only changes properties. Keep effects like these on the client, too: the docs point out that the server only needs a blast's position to decide its outcome, while clients create the visuals.

`Debris` is still right for a rare one-off, like a door that shatters once. For effects that fire many times a second, reuse.

**Key points**

- The engine waits for `PreRender` handlers before rendering; keep them for what truly changes every frame.
- Drive everything else from events, like `HealthChanged`, instead of polling each frame.
- A change inside a ScreenGui makes it recompute, so give fast-changing UI its own ScreenGui.
- Pool effects made many times a second: create once, hide, reuse. Debris suits rare one-offs.
- Create visual effects on the client; the server only needs what decides the outcome.
