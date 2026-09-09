# Sources

*A source stores one value; call it to read, call it with a value to write.*

Vide · Sources, Effects & Scopes · 9 min

---

A **source** is Vide's unit of state. `source(value)` returns a function. Call it with no arguments to read the current value; call it with one argument to store a new value. That is the whole API, and it is why Vide code has no `.get()` or `:set()` clutter.

```luau
local source = vide.source

local count = source(0)

print(count())     -- 0
count(count() + 1) -- write: increment by 1
print(count())     -- 1
```

## Deriving from a source

Wrap a source in a function and you have a value that is always computed from the latest state. Vide calls this a **derived source**. For now it is just a function; in the last lesson of this module you will see `derive()`, which adds caching.

```luau
local count = source(0)

local text = function()
	return "count: " .. tostring(count())
end

print(text()) -- "count: 0"
count(1)
print(text()) -- "count: 1"
```

On its own this is no better than a variable. The payoff comes in the next lesson, when something *watches* the source.

## What counts as a change

Writing the value a source already holds is a no-op: nothing downstream reruns. Numbers, strings and booleans are compared with `==`. Tables are different: writing a table, even the one already stored, always counts as a change unless the table is frozen, because you probably mutated it. That is how list updates work: edit the table, then write it back.

**Updating a table in a source**

```luau
local items = source({ "sword", "shield" })

local list = items()
table.insert(list, "potion") -- nothing has noticed yet
items(list)                  -- now dependents rerun
```

Frozen tables (`table.freeze`) are the exception to the exception: an identical frozen table is treated as unchanged.

**Key points**

- `source(v)` returns a function: `s()` reads, `s(v)` writes.
- A derived source is any function that reads sources.
- Writing an equal value is ignored; writing the same table counts as a change, so mutate then write back.
