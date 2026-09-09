# Lerping to a goal

*Move to a goal over a fixed duration, once you have checked the type can be interpolated.*

Fluid · Animation & Timing · 11 min

---

Where a spring is physical, `lerp` is scheduled: give it a goal and a duration and it walks there in that time. Both arguments are `UsedAs`, so either can be a plain value or a reader — the duration can change mid-flight.

**Over one second**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local target = fluid.source(0)
	local eased = fluid.lerp(target, 1) -- one second

	target(100)
	step(30)
	print(math.floor(eased())) --> 49, about halfway
	step(30)
	print(math.floor(eased())) --> 100
end)

destroy()
```

Both arguments are `UsedAs`, so the duration itself can come from a source.

## Check the type before you reach for it

`lerp` interpolates by looking the value's type up in a table of interpolation functions. That table is short, it is not documented anywhere upstream, and it is not the same table `spring` uses.

| Type | `lerp` | `spring` |
| --- | --- | --- |
| `number` | Yes | Yes |
| **`UDim2`** — `Position`, `Size` | **No** | **Yes** |
| `UDim` | Yes | Yes |
| `Color3` | Yes | Yes |
| `Vector3` | Yes | Yes |
| `Vector2` | Yes | **No** |
| `Rect` | Yes | **No** |
| `CFrame` | No | No |

`UDim2` and `CFrame` are present in `lerp.luau` but commented out. `spring` raises `tried tweening <type>` for anything it cannot pack.

> **A `UDim2` lerp does not fail once — it fails on every frame of the animation**
>
> `lerp.step` looks the interpolation function up in a table and calls whatever it found, so a `UDim2` throws `attempt to call a nil value` from inside the animation step, on the next tick, from a stack frame with nothing to do with your code. It keeps throwing for the animation's whole duration — the entry is dropped from the active table when `t` reaches 1, which is also the frame the throw stops — and it re-arms on every goal change. Worse, the throw aborts `lerp.step` before `spring.step` runs, so **every other animation on screen stops advancing that frame too**.

**The way round it**

```luau
local destroy = fluid.root(function()
	local alpha = fluid.source(0)
	local eased = fluid.lerp(alpha, 0.4) -- a number: safe

	local frame = fluid.create("Frame")({
		Position = function()
			return UDim2.new(eased(), 0, 0.5, 0)
		end,
	})

	alpha(1)
end)

destroy()
```

Interpolate a number, assemble the `UDim2` in the binding. The property function reruns whenever `eased` changes.

That pattern generalises. Animate the *smallest* thing that has an interpolation function — an alpha, an offset, a transparency — and assemble the compound value in the property function. It is also cheaper: one animated node feeding several bindings, rather than one per property.

**Key points**

- `fluid.lerp(goal, seconds)` walks to the goal over a duration; both arguments may be readers.
- It interpolates `number`, `Vector2`, `Vector3`, `Color3`, `UDim` and `Rect`.
- `UDim2` and `CFrame` are commented out upstream and there is no guard — the failure is a throw on every frame the animation is active, and it takes `spring.step` down with it.
- `Position` and `Size` are `UDim2`: use `spring`, or lerp a number and build the `UDim2` in a binding.
- Animating one small value and deriving the rest is both safer and cheaper.
