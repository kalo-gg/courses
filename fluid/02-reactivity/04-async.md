# Async nodes

*A reactive node whose value arrives later, and a state that only ever says busy or ok.*

Fluid · The Reactive Graph · 12 min

---

Fetching a player's stats, waiting on a remote, loading an asset: the value is not there when the interface first draws. `fluid.async` makes that a first-class node. Fluid's README lists it first under "the interesting bits where Fluid has an advantage", and it is one of the better reasons to pick the library — though that comparison is the project's own claim, not something this course verified against Vide and Fusion.

**A value and a state**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local stats, state = fluid.async(function(set)
		-- in a game this would yield on a remote
		return { wins = 12 }
	end)

	print(state())              --> "busy" until it resolves, then "ok"
	print(stats() and stats().wins)
end)

destroy()
```

Two readers come back: the value, and the state of the fetch.

> **There is no failure state**
>
> `State` is exactly `"busy" | "ok"` — that is the whole type. If your callback errors, Fluid logs a warning and the state still becomes `"ok"`, with the value left as it was. If a failed load needs to show something, catch it inside the callback and set a value that says so.

## What `set` is for

The callback's one parameter is a function that pushes a value out **without returning**. Returning from the callback is the same as calling `set` one last time, so for a single fetch you can ignore it — but `set` is what lets a long-running callback publish as it goes, so the interface can show partial results instead of a spinner.

**Streaming partial results**

```luau
local function fetchRow(index)
	-- in a game this would yield on a remote
	return "row " .. index
end

local destroy = fluid.root(function()
	local page, state = fluid.async(function(set)
		local rows = {}

		for index = 1, 5 do
			table.insert(rows, fetchRow(index)) -- this is allowed to yield
			set(table.clone(rows))              -- publish what we have so far
		end

		return rows
	end)

	fluid.effect(function()
		print(#(page() or {}), "rows so far,", state())
	end)
end)

destroy()
```

Every `set` invalidates the value node, so anything reading `page()` redraws. The final `return` is just the last update.

## Yielding is the whole point

Every other reactive callback in Fluid is forbidden from yielding. The async callback is the exception, and it is the reason the primitive exists: Fluid runs it on its own thread with `task.spawn`, so a `:InvokeServer()` or a `:WaitForChild()` inside it is fine.

> **"Always deferred" is about the callback, not the node**
>
> A comment in `async.luau` says evaluations are deferred because the callback "yields every single time", which reads as if the node waits for a flush. It does not: its evaluation mode is `"async"`, which the graph groups with **eager**, so it is invalidated straight away. What is deferred is the callback, on its own thread — and the graph never caches its return value, so what you read comes from `set`.

> **In this sandbox, async looks instant**
>
> There is no real yielding here, so a callback that would wait on a remote returns immediately and the state reads `"ok"` straight away. In a game you would see `"busy"` first. The API is identical either way; only the timing is compressed.

**Key points**

- `fluid.async(fn)` returns a value reader and a state reader; the state is only ever `"busy"` or `"ok"`.
- There is no failure state — catch errors inside the callback and put the failure in the value.
- The callback is the one place in Fluid that may yield; it runs on its own thread.
- `set(value)` publishes without returning, so a long callback can show partial results; the final `return` is one last `set`.
- The node itself is invalidated eagerly, not deferred; only the callback is deferred.
