# Springs

*Two return values: where the value is now, and a control function for pushing it around.*

Fluid · Animation & Timing · 11 min

---

`fluid.spring` takes a goal, a speed and a damping, and returns **two** things: a reader for the current value, and a control function. Missing the second one is the most common mistake for anyone arriving from Vide, and it fails quietly.

**The pair**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local goal = fluid.source(0)
	local position, control = fluid.spring(goal, 25, 1)
	--    ^^^^^^^^  ^^^^^^^
	--    a reader   a control function

	goal(100)
	step(60) -- one second in this sandbox
	print(math.floor(position())) --> 100
end)

destroy()
```

Write `local position = fluid.spring(...)` and you silently drop the controls.

## The control table

The control function takes a **table**, not positional arguments. Three keys are accepted, and you can pass any combination of them. Each one answers a different question, and none of them touches the goal — the goal is still whatever you wrote into the input source.

| Key | Effect |
| --- | --- |
| `position` | Teleport the value, leaving the goal alone — so it springs back. |
| `velocity` | Set how fast it is currently moving. |
| `impulse` | Add to its velocity: a shove rather than an assignment. |

**Flying in from off-screen**

```luau
local destroy = fluid.root(function()
	local goal = fluid.source(0)
	local offset, control = fluid.spring(goal, 25, 1)

	-- start it somewhere else; the goal is still 0, so it travels back
	control({ position = -200 })

	-- or give it a shove without moving it
	control({ impulse = 40 })

	step(60)
	print(math.floor(offset()))
end)

destroy()
```

`position` says where it is. The goal says where it wants to be. They are different questions. `spring` creates a graph node, so it needs the root.

> **The reader does not move until the next step**
>
> `control({ position = -200 })` writes into the spring's internal state; the readable you got back is only updated in the animation phase of a `step`. So reading the position on the line after `control` gives you the *old* value. Step once, then read.

| Argument | What it means | Useful range |
| --- | --- | --- |
| `speed` | The spring's angular frequency, in radians per second — it appears as `exp(-t × speed × damping)`, so doubling it roughly halves the settling time. | 20–30 feels right for UI; 5 is visibly sluggish, 60 lands within a frame |
| `damping` | A **ratio**, not a coefficient. `1` is critical damping: the fastest approach that never crosses the goal. | `1` for menus and tooltips; `0.5`–`0.8` for a deliberate bounce |

At `speed = 25, damping = 1` a 0→100 spring reaches 92 after ten frames and 100 after twenty. At `damping = 0.5` it overshoots to about 114 first. Avoid damping above 1: upstream passes `speed` and `damping` to its over-damped solver in the wrong order, so `damping = 2` crawls — 2.5 out of 100 after a full second.

> **Not everything is springable**
>
> `spring` handles `number`, `Vector3`, `Color3`, `UDim` and `UDim2` — including `Position` and `Size`, which is the important one. Anything else raises `tried tweening <type>` the moment you create it, so `Vector2` and `Rect` are out. The next lesson has the full matrix, because `lerp`'s list is *different*.

One spring per animated value, created once and kept. A spring cannot change data type after it is built — the assertion is `Can't switch the data types of a spring` — so a value that is sometimes a number and sometimes a `UDim2` needs two springs and a `show` between them, not one clever one.

**Key points**

- `fluid.spring(goal, speed, damping)` returns `(reader, control)`. Capture both.
- `control` takes a table: `position` teleports, `velocity` sets, `impulse` adds — and none of them change the goal.
- The reader catches up on the next `step`, not on the line after `control`.
- `speed` is roughly radians per second; `damping = 1` is critical damping, the fastest approach with no overshoot.
- Springable types: `number`, `Vector3`, `Color3`, `UDim`, `UDim2`. Anything else raises `tried tweening <type>` at creation.
