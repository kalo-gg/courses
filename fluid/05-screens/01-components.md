# Components are functions

*There is no component API: a component is a function that returns what create returned.*

Fluid · Building Real Screens · 11 min

---

There is no component API in Fluid. A component is a function that takes whatever it needs and returns what `create` returned. Because properties accept sources and functions alike, props are ordinary arguments.

**A button**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local function Button(props)
	return fluid.create("TextButton")({
		Text = props.text,
		Size = props.size or UDim2.fromOffset(120, 40),

		-- a signal-valued property: Fluid connects it, and disconnects
		-- it when the surrounding scope is destroyed
		Activated = props.onClick,
	})
end

local destroy = fluid.root(function()
	return fluid.create("Frame")({
		Button({ text = "Buy", onClick = function() print("bought") end }),
		Button({ text = "Cancel" }),
	})
end)
```

`props.text` may be a string or a source: a plain value is assigned once, a function becomes a live binding. `Activated = nil` on the Cancel button is simply skipped, so an optional handler needs no guard.

> **Prefer the property to an action for events**
>
> It is tempting to reach for `fluid.action(function(inst) inst.Activated:Connect(...) end)`. Fluid does not disconnect what *you* connect inside an action, so that pattern leaks a connection per button for the lifetime of the game. Naming the signal as a property — `Activated = fn` — hands the connection to the scope, which disconnects it on teardown. Keep `action` for things the property table genuinely cannot express.

> **Let the caller own the scope**
>
> A component should not call `fluid.root` itself. Its reactive nodes belong to whatever root the caller is already inside, which is what lets a whole screen's bindings, effects and connections be stopped in one call. Reach for a nested root only when a piece genuinely outlives or predeceases its parent.

## Props are just arguments

A prop can be a plain value or a reader, and the component does not need to know which — `Text = props.text` works either way, because `create` decides per value. That is the type Fluid calls `UsedAs<T>`: "a `T`, or a function returning one". Take props as `UsedAs` wherever you can and the caller chooses whether the value is live.

**One component, two kinds of caller**

```luau
local function Stat(props)
	return fluid.create("TextLabel")({
		Name = props.name,
		Text = props.value, -- a string, a source, or any function
	})
end

local destroy, panel = fluid.root(function()
	local score = fluid.source(0)

	local panel = fluid.create("Frame")({
		Stat({ name = "Mode", value = "Deathmatch" }), -- assigned once
		Stat({ name = "Score", value = score }),       -- a live binding
	})

	score(40)
	return panel
end)

print(panel.Mode.Text, panel.Score.Text) --> Deathmatch  40
destroy()
```

Neither call is special-cased. The plain string is written once; the source becomes a reactive node.

A component returns whatever the caller can use as a child: one instance, or an array of them. It does **not** open its own `root` — the caller's scope owns everything it builds, which is what lets a whole screen be stopped with one destructor.

**Key points**

- A component is a function. There is no registration, no lifecycle and no base class.
- Props are arguments; a prop may be a value or a reader, and `create` decides per value.
- `Activated = props.onClick` with no handler is simply skipped.
- A component should not call `fluid.root` — the caller's scope owns what it builds.
- Connect events by naming the signal as a property; keep `action` for what the table cannot express.
