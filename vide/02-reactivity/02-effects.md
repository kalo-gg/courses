# Effects

*Effects run when the sources they read change; untrack reads a value without subscribing.*

Vide · Sources, Effects & Scopes · 10 min

---

An **effect** is a function that Vide runs once immediately and then again every time a source it read is written. Think of a source as a signal and an effect as a connection, except you never connect anything: Vide notices which sources the function reads while it runs and subscribes to them for you.

```luau
local source = vide.source
local effect = vide.effect
local root = vide.root

local count = source(0)

root(function()
	effect(function()
		print("count: " .. count())
	end)
end)
-- "count: 0" printed immediately

count(1) -- "count: 1"
count(1) -- nothing: the value did not change
count(2) -- "count: 2"
```

`root` is needed because an effect must be created inside a scope. The next lesson explains why; for now, treat it as the box effects live in.

## Dependencies are discovered by reading

Tracking follows reads wherever they happen. If the effect calls a derived function that reads a source three calls deep, the effect still reruns when that source changes. Only sources that are actually *read during the run* are tracked, so a source behind an `if` that was not taken is not a dependency until it is.

**Dependencies through derived functions**

```luau
local count = source(1)

local doubled = function()
	return count() * 2
end

root(function()
	effect(function()
		print("doubled: " .. doubled())
	end)
end)
-- "doubled: 2"
count(2)
-- "doubled: 4"
```

## Reading without tracking

Sometimes an effect needs a value but should not rerun when it changes. `untrack(fn)` runs `fn` outside the current tracking, so any sources it reads are invisible to the effect.

```luau
local untrack = vide.untrack

local a = source(0)
local b = source(0)

root(function()
	effect(function()
		print(`a: {a()} b: {untrack(b)}`)
	end)
end)

a(1) -- prints "a: 1 b: 0"
b(1) -- prints nothing
a(2) -- prints "a: 2 b: 1"
```

`untrack(b)` works because `b` is itself a function: `untrack` calls it and returns the value.

**Key points**

- `effect(fn)` runs `fn` immediately and again whenever a source it read changes.
- Dependencies are discovered by reading, at any depth of function calls.
- Writing an unchanged value does not rerun effects.
- `untrack(source)` reads a value without subscribing to it.
