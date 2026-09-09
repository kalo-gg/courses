# Implicit effects

*A function as a property or child value becomes an effect that keeps it in sync.*

Vide · Reactive UI · 10 min

---

Writing an explicit effect for every property gets old fast, so Vide lets the property table do it. When a **non-event** string key gets a function, `create` wraps it in an effect that assigns the function's result to the property and reruns when its sources change. The counter becomes one expression.

```luau
local function Counter()
	local count = source(0)

	return create "TextButton" {
		Activated = function()   -- event: connected, not bound
			count(count() + 1)
		end,

		Text = function()        -- property: bound with an implicit effect
			return "count: " .. count()
		end,
	}
end
```

Vide tells the two apart by checking whether the instance's property is an event (`RBXScriptSignal`).

> **Only functions are reactive**
>
> `Text = count()` reads the source **once**, at creation, and assigns a plain number. Nothing updates later. To bind a source directly you can write `Text = count` (the source is a function) or wrap it: `Text = function() return count() end`.

A source is already a function, so it can be bound without a wrapper: `Visible = full` binds `Visible` to the source `full`, and a derived source binds the same way. Wrap only when the property needs a computation, such as `Text = function() return "HP " .. hp() end`. Either way, what you hand Vide is a function it can call again.

## Children as functions

A function in a **numeric** slot is a child binding. It may return one instance, an array of instances (nested arrays are fine) or `nil`. Vide creates an effect that parents whatever the function returns and unparents instances that disappear between runs. That is how a list source turns into a list of rows.

**A list bound to a source**

```luau
-- inside a root or component
local items = source {
	create "TextLabel" { Text = "A" },
}

local function List(props: { children: () -> { Instance } })
	return create "Frame" {
		create "UIListLayout" {},
		props.children,       -- child binding
	}
end

local list = List { children = items } -- contains label A

items {
	create "TextLabel" { Text = "B" },
	create "TextLabel" { Text = "C" },
}
-- A is unparented; B and C are parented
```

Dynamic Scopes introduces `indexes` and `values`, which build these child arrays for you from plain data. Your first project builds one by hand first.

**Key points**

- A function on a non-event property key is an implicit effect that keeps the property current.
- A plain value, even `count()`, is copied once and never updates.
- A function in a child slot binds children: returned instances are parented, missing ones unparented.
