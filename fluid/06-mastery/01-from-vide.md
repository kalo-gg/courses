# Porting a Vide screen

*What a Vide screen must change to become a Fluid screen, and what breaks without a word.*

Fluid · Mastery & Migration · 12 min

---

Fluid is, in its own README's words, "heavily based off Vide" with "full (opinionated) rewrites and significant breaking changes". If you have a Vide screen, most of it ports mechanically. What makes the port worth planning is the handful of places where the same-looking code compiles, runs, and does something else.

| Vide | Fluid | Note |
| --- | --- | --- |
| `source(0)` | `fluid.source(0)` | Same |
| `derive(fn)` / `effect(fn)` | `fluid.derive(fn)` / `fluid.effect(fn)` | Same, plus a `previous` argument |
| `root(fn)` | `fluid.root(fn)` | Same; destructor first in both |
| `create "Frame" { … }` | `fluid.create("Frame")({ … })` | Curried in both; Fluid uses parentheses |
| `mount(fn, inst)` | `fluid.mount(fn, inst)` | Same shape |
| `cleanup(x)` | `fluid.cleanup(x)` | Same |
| `untrack(fn)` / `read(v)` | `fluid.untrack(fn)` / `fluid.read(v)` | Same |
| `show(cond, fn)` / `switch(cond)(map)` | `fluid.show(…)` / `fluid.switch(cond)(map)` | Fluid's branches may be values as well as functions |
| `values(list, fn)` | `fluid.for_values(list, fn)` | **Different transform** |
| `indexes(list, fn)` | `fluid.for_keys(list, fn)` | **Different transform** |
| `spring(source, period)` | `fluid.spring(goal, speed, damping)` | **Different arguments and a second return value** |
| `batch(fn)` | — | No equivalent; reach for `fluid.deferred` on the reader's side |
| `context()` | — | No equivalent; thread values through props |
| `apply(inst, props)` | `fluid.bind(inst, props)` | Renamed |
| `vide.strict` | — | Dev mode is always on and is not configurable |

## The four that break silently

Four differences produce running code that is wrong. None of them errors, and three of them are invisible until someone looks at the screen: the missing defaults, the dropped spring control, the `for_values` transform, and a `bind` with its arguments the wrong way round. Check for those four first and the rest of a port is mechanical.

**The Vide original**

```luau
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create, values, spring = vide.create, vide.values, vide.spring

local function Panel(items)
	local offset = spring(source(0), 0.3)

	return create "Frame" {
		Position = function()
			return UDim2.new(0, offset(), 0.5, 0)
		end,

		values(items, function(item, index)
			return create "TextLabel" { Text = item, LayoutOrder = index }
		end),
	}
end
```

Four things in this block change. Two of them would still run in Fluid.

**The Fluid port**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local function Panel(items)
	local goal = fluid.source(0)
	local offset, control = fluid.spring(goal, 25, 1) -- speed and damping, and a pair

	return fluid.create("Frame")({
		BorderSizePixel = 0,                -- Vide set this for you; Fluid does not
		BackgroundColor3 = Color3.new(1, 1, 1),

		Position = function()
			return UDim2.new(0, math.floor(offset()), 0.5, 0)
		end,

		fluid.for_values(items, function(item, key, active) -- key is a reader now
			return fluid.create("TextLabel")({ Text = item, LayoutOrder = key })
		end),
	})
end
```

`spring` gained a control function and lost its period; `values`' index became a reader; the defaults had to be written out.

> **The port that looks finished and is not**
>
> Fluid applies no per-class defaults. A Vide `Frame` arrives with `BorderSizePixel = 0` and a white background; the Fluid one does not. Nothing errors — the screen simply comes back with Roblox's gray panels and one-pixel borders. Write the defaults out, or into a shared style table.

Do the port in one direction: sources and derives first, then instances, then lists, then animation. Each stage runs on its own, and each stage's failure mode is distinct, so a mistake is easy to place. Leave the styling pass for last, when the tree is right.

**Key points**

- Most of a Vide port is renames: the `source` / `derive` / `effect` / `root` spine is unchanged.
- `values` → `for_values` and `indexes` → `for_keys`, both with a different transform.
- `spring` takes a speed and a damping and returns a **pair**; dropping the control is silent.
- `apply` → `bind`, with the instance first, and the wrong order writes `nil` into your source.
- `batch`, `context` and `vide.strict` have no equivalent, and Fluid sets no per-class defaults.
