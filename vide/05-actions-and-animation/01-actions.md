# Actions

*Attach a callback to a property table; it runs with the finished instance.*

Vide · Actions, Springs & Context · 8 min

---

An **action** is a callback you put in a property table's unkeyed slots. When `create` has set the properties and connected the events, it calls each action with the instance. Actions are how you reach the instance from inside its own description: to store a reference, measure it, or wire it to something outside Vide.

```luau
local action = vide.action

create "TextLabel" {
	Text = "test",

	action(function(instance)
		print(instance.Text) -- "test": properties are already applied
	end),
}
```

## Priorities and reusable actions

Actions take an optional **priority**; the default is 1 and lower numbers run first in practice (Vide iterates its priority map without sorting, which orders small positive priorities but not 0, negative or sparse ones). Because an action is a value, you can wrap it in a function and reuse it. A common one is a *ref* that stores the instance in a source so other code can reach it.

**A reusable ref action**

```luau
local function ref(target: (Instance) -> ())
	return action(function(instance)
		target(instance)
	end)
end

local frame = source(nil :: Frame?)

create "Frame" {
	ref(frame),
}

print(frame().ClassName) -- "Frame"
```

## apply: the same rules on an instance you already have

`apply(instance)(props)` runs the property-table rules — properties, events, children, actions, `Parent` last — on an instance that already exists: a clone, or one built in Studio. It returns the instance. The source marks it *temporary* in 0.4.1, so use it at the edges (styling a clone, attaching actions to a Studio-made frame) rather than as a second way to build UI.

**apply**

```luau
local apply = vide.apply

local frame = Instance.new("Frame")

apply(frame) {
	Name = "Applied",
	create "UICorner" {},
	action(function(instance)
		print(instance.Name, #instance:GetChildren()) -- "Applied 1"
	end),
}
```

**Key points**

- `action(fn, priority?)` runs `fn(instance)` after properties and events are applied.
- Actions are plain values: wrap them in functions to make reusable helpers like `ref`.
- Lower priorities run first for small positive numbers; the default is 1.
- `apply(instance)(props)` applies a property table to an existing instance; it is marked temporary.
