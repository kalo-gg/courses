# Creating instances

*One mixed table holds the properties and the children, and Fluid sets no defaults of its own.*

Fluid · Fluid Foundations · 12 min

---

`fluid.create` is curried: the class name, then a table. That table is *mixed* — named keys are properties, and the array part is children. Fluid lists this as "mixed-table approach to properties & children" in its overview. It is not, despite how it reads, a departure from Vide: Vide's `create` is curried the same way and treats number keys as children too.

**Properties and children together**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local title = fluid.source("Shop")

	return fluid.create("Frame")({
		Name = "Panel",

		fluid.create("TextLabel")({
			Text = title, -- a source works straight through
		}),
		fluid.create("TextButton")({
			Text = "Close",
		}),
	})
end)
```

`Name` is a property because it has a key; the two children have none, so they are children.

> **Fluid applies no defaults**
>
> Vide sets per-class defaults before your properties — a `Frame` gets `BorderSizePixel = 0` and a white background, an `ImageButton` gets `AutoButtonColor = false`. Fluid applies **none**; its README lists "No defaults (Causes friction with stylesheets)" as a design choice. Port a Vide screen and Roblox's own defaults come back. Nothing errors; it just looks wrong.

A property accepts a plain value, a source, or any function returning a value — but the two cases are handled quite differently. A **plain value** is assigned once and forgotten. A **function** becomes a live subscription: Fluid creates a reactive node that re-assigns the property whenever anything the function read changes. That is why `Text = title` keeps up with the source instead of freezing at its first value.

## Events are properties too

If the name you use is a `RBXScriptSignal` on that class and the value is a function, `create` connects it — and registers the disconnect on the surrounding scope, so it goes when the scope does. You do not need a separate helper to handle a click.

**A button that counts**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy, button = fluid.root(function()
	local count = fluid.source(0)

	return fluid.create("TextButton")({
		Text = function()
			return "Clicked " .. count() .. " times"
		end,

		Activated = function()
			count(count() + 1)
		end,
	})
end)
```

`Text` is a property, so it becomes a binding. `Activated` is a signal, so it becomes a connection Fluid owns.

> **A non-function on a signal is silently ignored**
>
> The check is "is the current value of this property an `RBXScriptSignal`?" first, and "is the value a function?" second. `Activated = someHandler` where `someHandler` is a table — a Signal object, a Janitor wrapper — matches the first test and fails the second, so Fluid drops it on the floor without a word. Nothing is connected and nothing complains.

**`create` is on the list too, conditionally**

```luau
local label = fluid.source("hi")

-- fine anywhere: every property is a plain value
fluid.create("TextLabel")({ Text = "static" })

-- needs a root: a function property becomes a reactive node
local ok, err = pcall(function()
	return fluid.create("TextLabel")({ Text = label })
end)
print(err) --> cannot call ? without a parent scope
```

This is why `create` can look scope-free right up until the moment you make something dynamic.

**Key points**

- `fluid.create("Class")({ … })` is two calls: the class name, then one mixed table.
- Keyed entries are properties; the array part holds children and actions.
- A plain value is assigned once; a function becomes a live binding.
- A function on a signal-valued property is connected and disconnected with the scope; a non-function there is dropped in silence.
- Fluid sets no per-class defaults, and `create` needs a stable scope as soon as any property is a function.
