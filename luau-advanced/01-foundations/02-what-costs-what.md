# What costs what

*A rough price list for the operations in a loop body, and why allocation is the one that bites.*

Luau Advanced · How Luau Runs · 10 min

---

"Is this slow?" is unanswerable without a sense of scale. What follows is not a benchmark — this sandbox has no clock — but the ordering the VM's design implies, cheapest first. Use it to decide what is worth looking at, not to predict a number.

| Operation | What the VM does | Roughly |
| --- | --- | --- |
| Read a local | Read a register | Free |
| Read a library chain (`math.max`) | Resolved once when the script is loaded | Free, like a constant |
| Read an upvalue | One indirection into the closure | Nearly free |
| Read a table field | Hash or array lookup on the table | Cheap |
| Read a global you assign in this script | Hash lookup in the environment table | A table field, every read |
| Call a method through `__index` | Miss on the table, then a lookup in the metatable's `__index` | Two lookups plus a call |
| Build a string in a loop | A fresh string allocation per concatenation | The expensive one |

Ordered by the work the VM performs, not by a measurement. Nothing in this course was timed.

**Three ways to sum a list**

```luau
local list = { 1, 2, 3, 4, 5 }

local function byIndex(): number -- one array read per iteration
	local total = 0
	for i = 1, #list do
		total += list[i]
	end
	return total
end
local function byPair(): number -- the same, with the iterator supplying the value
	local total = 0
	for _, v in list do
		total += v
	end
	return total
end
local function byCall(): number -- plus a call per iteration, for nothing
	local total = 0
	for _, v in list do
		total = math.max(total, 0) + v
	end
	return total
end

print(byIndex(), byPair(), byCall())
```

All three print `15`. The third does strictly more work per iteration than the other two and buys nothing with it — but the extra work is the *call*, not a lookup: `math.max` was resolved when the script loaded, and builtins like it are called through a fast path that skips the usual frame.

## Allocation is the one that bites

The first six rows of that table differ by small constant factors. The last one is different in kind. A table, a closure or a concatenated string created inside a loop is memory the collector will have to walk and free later — and *later* is the problem. The frame that allocates is not the frame that stalls, which is exactly why allocation-driven hitches are so hard to trace back to their cause.

**The allocation you did not mean to write**

```luau
local handlers = {}

for i = 1, 3 do
	-- A brand-new closure every iteration, each capturing its own `i`.
	handlers[i] = function()
		return i
	end
end

print(#handlers, handlers[1](), handlers[3]())
```

Prints `3	1	3`. Three closures is fine. Three thousand, once per frame, is a garbage-collection problem that will show up somewhere else entirely.

The fix for an allocation inside a loop is almost always to **hoist it out**: create the object once, before the loop, and reuse it. That is only safe when nothing keeps a reference to it after the iteration ends — which is worth checking, because it is the one way this transformation can change behaviour rather than just cost.

**Key points**

- Locals are free, table and global reads are cheap, metamethod dispatch costs a call, allocation costs the collector.
- A global *you assign in this script* is a table lookup on every read, and hoisting it into a local removes that. A library chain like `math.max` is resolved when the script is loaded, so hoisting it removes nothing.
- Tables, strings and closures allocate. Numbers, booleans and vectors do not.
- The frame that allocates is rarely the frame that stalls, which is what makes allocation bugs hard to find.
