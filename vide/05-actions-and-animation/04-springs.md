# Springs

*Animate any source with spring physics; tune period and damping; nudge with the control.*

Vide · Actions, Springs & Context · 11 min

---

`spring(source, period, damping)` returns a new source whose value chases the input with spring physics. Bind a property to the spring instead of the raw source and it animates for free. Numbers, `UDim`, `UDim2`, `Vector2`, `Vector3`, `Color3`, `CFrame` and `Rect` are all supported.

```luau
-- inside a root or component
local spring = vide.spring

local hovered = source(false)

local size = spring(function()
	return if hovered() then UDim2.fromOffset(220, 60) else UDim2.fromOffset(200, 50)
end, 0.3)

create "TextButton" {
	Size = size,            -- animates between the two sizes
	MouseEnter = function() hovered(true) end,
	MouseLeave = function() hovered(false) end,
}
```

Any function works as the input: a plain source, a derived source, or a function that computes the target from state as above. The output is a new source, so you bind it exactly like any other, and it accepts every type the table below lists.

## Tuning

| Parameter | Meaning | Default |
| --- | --- | --- |
| `period` | Seconds for one full cycle if undamped; smaller is snappier, but `damping / period` above about 19 is rejected | 1 |
| `damping` | 1 = critically damped (no overshoot); < 1 overshoots and wobbles; > 1 is sluggish; 0 oscillates forever | 1 |

## The control, and stepping time

Springs advance every frame from `RunService.Heartbeat`, so in Roblox they just work. The Kalo sandbox has no frame loop; call `step(frames)` to advance time (60 frames is one second). The spring also returns a second value, a **control** function: call it with `{ position = v }` to move the solver (visible from the next step), `{ velocity = v }` to set speed, or `{ impulse = v }` to add a kick. Calling the spring source itself with a value snaps it there and clears the velocity.

**Nudging a spring**

```luau
-- inside a root or component
local shake = source(0)
local offset, control = spring(shake, 0.4, 0.3) -- underdamped: wobbles

-- on damage:
control { impulse = 40 } -- kick it; it springs back to 0 with a wobble
```

> **Time only moves when you step it, and stepping is one-way**
>
> In Roblox, springs advance on `RunService.Heartbeat`. The Kalo sandbox has no frame loop, so a spring's value stays put until you call `step(frames)`; sixty frames is one second, and forgetting to step is the usual reason a sandbox spring looks stuck. `step()` wraps the real `vide.step(dt)`, a public function that advances the spring solver and the delayed-destruction timers. Its first call **permanently disconnects** Vide's own Heartbeat connection, so from then on nothing animates unless you keep stepping: use it in test harnesses, never in a place file.

**Key points**

- `spring(source, period, damping)` returns an animated source; bind properties to it.
- Period sets speed, damping sets bounce: 1 is smooth, below 1 overshoots.
- The second return value controls position, velocity and impulses; the sandbox needs `step()` to advance.
- `step()` wraps the real `vide.step(dt)`. Calling it in Roblox takes the solver off Heartbeat for good, so save it for tests.
