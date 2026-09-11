# Callable tables and custom lengths

*__call makes a table a function, __len redefines #, and the table library ignores both.*

Luau Advanced · Metamethods in Full · 9 min

---

Two hooks change what a table *is* rather than what an operator does to it. `__call` makes it callable, so `M(x)` works. `__len` decides what `#M` reports.

The pair is how a module offers one obvious operation and still carries configuration. `formatter(value)` reads better than `formatter.format(value)`, and `formatter.locale` is still there when you need it.

**A table you can call, measure and print**

```luau
local M = setmetatable({ factor = 2 }, {
	__call = function(self, n: number)
		return n * self.factor -- self is the table, so configuration is reachable
	end,
	__len = function()
		return 7
	end,
	__tostring = function()
		return "M!"
	end,
})

print(M(21), #M, tostring(M))
```

Prints `42	7	M!`. Note `self`: `__call` receives the table as its first argument, so the call form still has access to the fields.

## __len and the array part

`__len` overrides `#`, and that is *all* it overrides. Two things go on ignoring it:

- `rawlen(t)` deliberately bypasses it, which is how you ask for the real array length.
- The **`table` library** does not consult it at all. `table.insert`, `table.remove` and `table.concat` all work on the raw array part.

So a table with `__len` returning 7 can genuinely hold two array entries, and `table.insert` will put the third at index 3 while `#t` keeps saying 7. That is not a bug to work around; it is the reason `__len` belongs on a wrapper type whose storage is somewhere else, not on a table people also treat as a list.

**What ignores __len**

```luau
local t = setmetatable({ 1, 2 }, {
	__len = function()
		return 7
	end,
})

print(#t, rawlen(t)) -- the hook, then the truth
print(table.concat(t, ","))

table.insert(t, 3)
print(rawlen(t), #t)
```

Prints `7	2`, then `1,2`, then `3	7`. `table.insert` appended at index 3, exactly as if `__len` did not exist.

`__tostring` belongs in the same group for a different reason: it costs nothing until something prints the value, and it turns every future error message and debug line from a table address into something readable. Of all the metamethods in this module it is the one with the best ratio of effort to payoff, and the one most often left out.

**Key points**

- `__call(self, ...)` makes a table callable and still hands it its own fields.
- `__len` overrides `#` and nothing else.
- `rawlen` bypasses it deliberately, and the whole `table` library ignores it.
- So `__len` belongs on a wrapper whose real storage is elsewhere, not on a table also used as a list.
