# The cost you cannot see

*Tables, closures and strings allocate; the collector charges for them later, and elsewhere.*

Luau Advanced · Shipping Fast Luau · 10 min

---

Of everything in Module 1's price list, allocation is the only entry whose bill arrives somewhere else. A table created in a loop costs a little to make and an unknown amount to collect, at a moment nobody chose, in a frame that had nothing to do with it.

That delay is the whole difficulty. A profile points at the collector, the collector points at nothing in particular, and the code responsible is three systems away.

**Three things that allocate**

```luau
local before = gcinfo() -- live heap, in whole kilobytes

local t = {} -- a table
local f = function() -- a closure, capturing t
	return t
end

local s = ""
for _ = 1, 1000 do
	s ..= "x" -- a brand-new string every single time
end

print(#s, f() == t, gcinfo() >= before)
```

Prints `1000	true	true`. The comparison is deliberate: the exact kilobyte figure depends on the host, the direction does not.

Three kinds of value are heap objects: **tables**, **closures** and **strings**. Numbers, booleans, `nil` and — importantly for this course — **vectors** are values that live in registers and cost nothing to create.

The string case is the sneakiest, because it hides inside an operator. `s ..= "x"` in a loop of a thousand builds a thousand strings, each one a copy of the last plus a character. `table.concat` on a list of pieces builds one.

## Reuse instead

There are three moves, in rough order of how often they apply.

**Hoist**: create the object once, above the loop, and write into it each iteration — Module 1's exercise. **Clear and refill**: `table.clear(t)` empties a table without allocating a replacement, so a scratch table can serve a whole run. **Size once**: `table.create(n)` gives an array its final capacity immediately, instead of letting it grow by reallocation as you fill it.

And the fourth, which is the previous three modules' whole point: if the data is numbers of a fixed shape, a **buffer** removes the allocation rather than reusing it.

**One table, reused**

```luau
local scratch = table.create(4) -- sized once, never regrown

local function sumInto(out: { number }, a: number, b: number): { number }
	table.clear(out) -- empty it without allocating a new one
	out[1] = a
	out[2] = b
	out[3] = a + b
	return out
end

for i = 1, 3 do
	local r = sumInto(scratch, i, i * 2)
	print(r[1], r[2], r[3])
end
```

Prints `1	2	3`, `2	4	6`, `3	6	9`. One table for the whole loop — safe only because nothing keeps `r` after the iteration ends.

**Key points**

- Tables, closures and strings allocate; numbers and vectors do not.
- The collector's bill arrives later and elsewhere, which is what makes allocation bugs hard to trace.
- Hoist, `table.clear` and `table.create` are the three reuse moves, in that order of frequency.
- For fixed-shape numeric data, a buffer removes the allocation rather than reusing it.
