# Derived values

*Cached values that recompute only when read, and only when the answer actually changed.*

Fluid · The Reactive Graph · 11 min

---

A **source** holds a value you write. A **derive** holds a value you compute. `fluid.derive(fn)` returns a readable, like a source, but you never write to it: it recomputes from whatever it read the last time something underneath it changed. Two properties make it worth reaching for instead of just calling a function.

**It runs when you read it**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local n = fluid.source(1)
	local runs = 0

	local doubled = fluid.derive(function()
		runs += 1
		return n() * 2
	end)

	print("before any read:", runs)   --> 0
	print(doubled(), runs)            --> 2  1
	print(doubled(), runs)            --> 2  1   (cached)

	n(5)
	print("after the write:", runs)   --> 1   (still nothing)
	print(doubled(), runs)            --> 10 2
end)

destroy()
```

A derive is a **lazy** node. The write marks it stale; the read is what pays for the recompute.

## The graph stops at equal values

A node only wakes its dependents when its value actually changed. Fluid's test is `a == b`, with one exception: two **mutable tables** are never treated as equal, even when they are the same table. So a derive that collapses many states into a few — a health number into `low` or `not low` — quietly saves every effect below it from re-running.

**An unchanged result goes no further**

```luau
local destroy = fluid.root(function()
	local hp = fluid.source(100)
	local low = fluid.derive(function() return hp() < 25 end)

	local runs = 0
	fluid.effect(function()
		low()
		runs += 1
	end)

	print(runs)  --> 1
	hp(80)
	print(runs)  --> 1   low is still false, so nothing downstream moved
	hp(10)
	print(runs)  --> 2   low flipped
end)

destroy()
```

Two writes, one rerun. The derive absorbed the first because its answer did not change.

> **A table always counts as changed**
>
> The equality test refuses to compare two mutable tables, so writing the *same table reference* back into a source still invalidates everything below it. If your list source is a table you mutate in place, every write is a full rebuild. Write a new table, or freeze the ones you compare.

## Reading without subscribing

Sometimes an effect needs a value but should not rerun when it changes — a threshold it compares against, a setting it reads once per run. `untrack` reads inside a callback without recording the dependency.

**untrack**

```luau
local threshold = fluid.source(50)
local health = fluid.source(100)

local destroy = fluid.root(function()
	fluid.effect(function()
		local limit = fluid.untrack(function()
			return threshold()
		end)

		if health() < limit then
			print("low health")
		end
	end)

	threshold(90) -- the effect does NOT rerun
	health(10)    -- it does
end)

destroy()
```

The effect depends on `health` alone, because that is the only source it read directly. Note the `root` — the sources are fine at the top level, the effect is not.

**Key points**

- `fluid.derive(fn)` returns a readable you call; you never write to it.
- It is **lazy**: a write marks it stale, a read recomputes it, and repeat reads are free.
- A node whose value did not change does not wake anything below it — but two mutable tables are never "unchanged".
- `fluid.untrack(fn)` reads inside a reactive callback without recording the dependency.
- `derive` needs a stable scope, and calling it from inside another reactive callback is the error `untrack` is named in.
