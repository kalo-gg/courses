# Frozen tables

*table.freeze is shallow and in place, table.clone is shallow and never frozen, and both matter.*

Luau Advanced · Buffers & Vectors · 9 min

---

A configuration table that anyone can write to is a configuration table that will eventually be written to, usually at a distance and usually by accident. `table.freeze` makes that impossible: it marks a table read-only, in place, and returns it. A later write raises instead of quietly drifting.

**freeze, isfrozen, and the error a write raises**

```luau
local config = table.freeze({ speed = 10, tags = { "fast" } })
print(table.isfrozen(config))

local ok, err = pcall(function()
	config.speed = 20
end)
print(ok, string.match(tostring(err), "attempt.*"))
```

Prints `true`, then `false	attempt to modify a readonly table`. The `string.match` trims the file-and-line prefix so only the message shows.

Two properties decide how you use it, and both are easy to assume wrongly.

`table.freeze` is **shallow**. It freezes the table you hand it and nothing inside it, so `config.tags` in the example above is still a perfectly ordinary, perfectly writable table.

`table.clone` is also **shallow**, and its result is **never frozen** — even when the source was. That is the intended way to hand out an editable copy of a frozen table, and it is also the hole in a sandbox built from freezing alone: the nested tables are shared, not copied.

**Shallow, both ways**

```luau
local config = table.freeze({ speed = 10, tags = { "fast" } })

table.insert(config.tags, "loud") -- the inner table was never frozen
print(#config.tags, config.tags[2])

local copy = table.clone(config)
print(table.isfrozen(copy)) -- a clone is never frozen
copy.speed = 20
print(copy.speed)
```

Prints `2	loud`, then `false`, then `20`. Freezing the outer table stopped nothing at all one level down.

## What it buys

Freezing buys two things. The first is that configuration cannot drift: a value read at frame 1 is the value at frame 6000, and nothing has to be defensive about it. The second is a guarantee a *reader* can rely on — code holding a frozen table can pass it around, cache values from it and skip copying it, because nobody can invalidate any of that.

That second point is why the final project's `config.luau` returns a frozen table and the run asserts it is still frozen at the end. It is not decoration; it is the reason the rest of the code can be written the way it is.

**Key points**

- `table.freeze` marks a table read-only in place; a later write raises `attempt to modify a readonly table`.
- It is shallow: nested tables stay fully writable until you freeze them too.
- `table.clone` is shallow as well, and its result is never frozen even when the source was.
- A frozen table is a promise a reader can build on: cache from it, share it, skip the defensive copy.
