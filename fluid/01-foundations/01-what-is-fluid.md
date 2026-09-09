# What is Fluid?

*Declarative UI for Roblox, built on a push-pull reactive graph that started life as Vide.*

Fluid · Fluid Foundations · 9 min

---

**Fluid** is a declarative UI library for Roblox. You describe what the interface looks like for the current state, and Fluid keeps the instances in step — the same bargain Vide and Fusion offer. Its own stated aims are correctness, simplicity and stability.

**The shape of it**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local count = fluid.source(0)

	return fluid.create("TextButton")({
		Text = function()
			return "Clicked " .. count() .. " times"
		end,
	})
end)
```

A source holds state, a function reading it becomes a binding, and `root` owns the lot.

## Its relationship to Vide

Fluid is, in the project's own words, "heavily based off Vide" — which is itself based on Solid — but with "full (opinionated) rewrites and significant breaking changes". If you have taken the Vide course here, you already know the spine: `source`, `derive`, `effect`, `root`. What differs is the detail, and the detail is where the bugs live.

| Idea | Vide | Fluid |
| --- | --- | --- |
| State | `source(0)` | `fluid.source(0)` |
| Derived value | `derive(fn)` | `fluid.derive(fn)` |
| Side effect | `effect(fn)` | `fluid.effect(fn)` |
| An element | `create "Frame" { … }` | `fluid.create("Frame")({ … })` |
| A list | `indexes` / `values` | `for_values`, whose transform also receives an `active` readable |
| Motion | `spring(source)` | `fluid.spring(goal, speed, damping)`, returning a **pair** |
| Property defaults | Per class: a `Frame` gets `BorderSizePixel = 0` and a white background | **None.** "No defaults (Causes friction with stylesheets)" |

The first four rows are syntax; the last is the one that will surprise you. A ported Vide screen comes back with Roblox's own borders and colors, and nothing in the code says why.

**The same component, both ways**

```luau
-- Vide
local function Counter(count)
	return create "TextButton" {
		Text = function()
			return "Clicked " .. count() .. " times"
		end,
	}
end

-- Fluid
local function Counter(count)
	return fluid.create("TextButton")({
		Text = function()
			return "Clicked " .. count() .. " times"
		end,
	})
end
```

The spine is identical and the call syntax is not. Luau's no-parens sugar passes one argument, so it cannot express a curried call — `create "Frame" { … }` becomes `fluid.create("Frame")({ … })`, with both sets of parentheses.

Underneath, Fluid's graph is **push-pull**. A write pushes an invalidation outwards to mark everything downstream stale; a read pulls the value back through, recomputing only what it has to. Some of that work is held for a deferred flush.

> **Which is why the scheduler has a step**
>
> Because some work waits for a flush, Fluid's frame loop is an explicit `step` you can take over — and that is exactly how the sandbox on this site advances it by hand.

**Key points**

- Fluid is declarative UI for Roblox: describe the interface for the current state, and Fluid keeps the instances in step.
- It is Vide's spine — `source`, `derive`, `effect`, `root` — with the details rewritten on purpose.
- A plain property value is assigned once; a function becomes a live binding.
- The graph is push-pull with a deferred phase, which is why the scheduler has a step you can drive.
- Familiarity with Vide is a hint, not a guarantee: check the signature before you paste.
