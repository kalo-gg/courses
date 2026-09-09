# Instances you did not build

*Bind onto a Studio template, read a property back, reach the instance itself, and tag it.*

Fluid · Building Real Screens · 12 min

---

Most of the time state flows into instances. Three helpers handle the cases where it flows the other way, or where you need the instance itself. Clicks are **not** on this list any more — those are just properties, as the lesson on `create` covered.

| Helper | For | Shape |
| --- | --- | --- |
| `changed` | Reading a Roblox property now and on every change | `fluid.changed("Text", function(value) … end)` |
| `bind` | Applying a props table to an instance you did not create | `fluid.bind(inst, { Text = source })` |
| `action` | Anything else needing the instance itself | `fluid.action(function(inst) … end)` |
| `tags` | Adding CollectionService tags that follow the scope | `fluid.tags({ "Hud", fn })` |

> **`bind` takes the instance first**
>
> It is `create`'s applier with the `Instance.new` step removed: `fluid.bind(inst, props)` takes the same mixed table, applies it, and returns the instance — `fluid.create("Frame")(props)` is literally `bind(Instance.new("Frame"), props)`. Getting the order wrong is worse than an error: `fluid.bind("Name", mySource)` type-checks at runtime as a generic `for` loop, Luau calls `mySource(nil, nil)` as an iterator, and a source called *with an argument* is a **write** — so your source is silently set to `nil`.

**Binding onto something from Studio**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local title = fluid.source("Shop")
	local existing = Instance.new("TextLabel") -- pretend this came from a ScreenGui

	fluid.bind(existing, {
		Text = title,
		fluid.create("UIPadding")({}),
	})

	print(existing.Text) --> Shop
	title("Inventory")
	print(existing.Text) --> Inventory
end)

destroy()
```

Same table, same rules: a function property becomes a live binding, and the array part still holds children and actions.

> **`changed` fires immediately, not only on change**
>
> It calls your function with the property's **current** value before connecting anything, then again on every change. And it runs *early*: actions execute while Fluid iterates the props table, so they can run before the hash-keyed properties in that same table are applied, and always before the instance has a `Parent`. Guard for that first call.

**An action**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	return fluid.create("TextBox")({
		fluid.action(function(inst)
			-- the instance exists, but is not finished: no Parent yet,
			-- and the properties below may not be applied
			print("action sees Text =", inst.Text, "Parent =", inst.Parent)
		end),

		fluid.changed("Text", function(newText)
			print("they typed", newText)
		end),

		Text = "start",
	})
end)

destroy()
--> action sees Text = nil Parent = nil
--> they typed nil
--> they typed start
```

Both go in the array part of the table, alongside children — Fluid tells them apart by what they are. Three lines of output from one build: in **this** run the action happened to go first and `changed` read the property before `Text = "start"` was applied, so it fired once with `nil` and once with the value. The order is not guaranteed — Luau leaves a mixed table's iteration order undefined — but the `Parent` is: both ran before the instance had one.

## Tagging what you build

`fluid.tags` is an action that adds CollectionService tags and removes them when the scope ends — so a system elsewhere can find your instances by tag without your screen having to register anything. An entry may be a plain string, or a **function** returning a string or `nil`, which makes the tag follow state.

**A tag that follows a source**

```luau
local destroy, frame = fluid.root(function()
	local open = fluid.source(true)

	return fluid.create("Frame")({
		fluid.tags({
			"Hud",
			function()
				return if open() then "Open" else nil
			end,
		}),
	})
end)

print(frame:HasTag("Hud"), frame:HasTag("Open")) --> true  false
step(1)
print(frame:HasTag("Open"))                      --> true
destroy()
print(frame:HasTag("Hud"))                       --> false
```

The plain tag is applied at once; the function entry becomes a **deferred** node, so it lands on the next step. Both are removed when the scope ends.

One rule ties them together: actions and children share the array part of the table, so Fluid tells them apart with `fluid.is_action(value)`. If you write a helper that returns an action, that is the function to identify it with.

**Key points**

- `fluid.bind(inst, props)` applies `create`'s mixed table to an instance you already have, and returns it.
- The instance comes **first**; the wrong order silently writes `nil` into your source.
- `fluid.changed(prop, fn)` calls you with the current value immediately, then on every change — often before the rest of the table is applied.
- `fluid.action(fn)` hands you the instance itself, half-built and unparented; anything you connect there is yours to disconnect.
- `fluid.tags` adds tags that follow the scope; a function entry is deferred, so it lands on the next step.
