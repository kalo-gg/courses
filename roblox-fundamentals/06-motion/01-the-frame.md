# The frame

*RunService fires events every frame; scale what you move by the frame's length and it runs at the same speed at any frame rate.*

Roblox Fundamentals · Motion · 10 min

---

A coin in the lobby should spin a quarter turn every second. The first version turns it `math.rad(1.5)` every frame, which is right at 60 frames a second. But frames are not all the same length: on a busy server they take longer, and a coin that turns a fixed amount per frame slows down with them.

**RunService** fires events once per frame, and each hands your handler `dt`, the length of that frame in seconds. Multiply a speed by `dt` and the motion is measured in seconds, not frames.

## Every frame, scaled by dt

**A crystal that bobs**

```luau
local RunService = game:GetService("RunService")

local crystal = workspace.Crystal
local start = crystal.Position
local elapsed = 0

RunService.Heartbeat:Connect(function(dt)
	elapsed += dt
	crystal.Position = start + Vector3.new(0, math.sin(elapsed) * 2, 0)
end)
```

`elapsed` adds up real seconds, not frames, so the crystal keeps the same rhythm whatever the frame rate. Anything that changes a little each frame is scaled by `dt` the same way.

Every frame runs in the same order. `PreSimulation` fires, physics steps the world, then `PostSimulation` fires. Last comes `Heartbeat`, which is also when threads waiting on the `task` library resume.

- **PreSimulation**: change what physics is about to use, such as a velocity.
- **PostSimulation**: react to where physics left things.
- **Heartbeat**: most other per-frame work, like the coin.

`PreRender` fires only on the client, so a server Script never uses it.

> **Recognising older code**
>
> Older scripts connect `RunService.Stepped` and `RenderStepped`. They still work and the API does not deprecate them, but the docs say `PreSimulation` and `PreRender` supersede them for new work. `Stepped` passes `(time, deltaTime)`, so a converted handler drops its first argument.

## The rest of the task library

The Events module used `task.wait`, `task.spawn` and `task.cancel`. Two more calls schedule work on the same frame clock without yielding. Each returns the thread it scheduled, so `task.cancel` can call that work off too.

| Call | Does | Returns |
| --- | --- | --- |
| `task.delay(seconds, fn, ...)` | Runs `fn` on the first Heartbeat after `seconds`, without yielding the caller | The thread |
| `task.defer(fn, ...)` | Runs `fn` at the end of the current resumption point, not immediately like `task.spawn` | The thread |
| `task.wait(seconds)` | Yields at least that long, then resumes on the next Heartbeat | The time that actually passed |

**Bring the bridge back later, unless the round ends first**

```luau
local bridge = workspace.Bridge

bridge.CanCollide = false
local restore = task.delay(3, function()
	bridge.CanCollide = true
end)

-- later, if the round ends before the 3 seconds are up:
task.cancel(restore)
```

`task.delay` returns at once with the thread it scheduled, and `task.cancel` stops it the way it stopped a per-part loop in the Events module. Keep the thread whenever something might need to call the work off.

**Key points**

- `PreSimulation`, `PostSimulation` and `Heartbeat` fire every frame and pass `dt`, the frame's length in seconds.
- Scale every per-frame change by `dt`, so the speed is the same at 30 and at 60 frames a second.
- A frame runs PreSimulation, physics, PostSimulation, then Heartbeat, when waiting threads resume.
- `PreRender` is client-only; `Stepped` and `RenderStepped` are the older names.
- `task.delay` and `task.defer` schedule work without yielding and return a thread that `task.cancel` can stop.
