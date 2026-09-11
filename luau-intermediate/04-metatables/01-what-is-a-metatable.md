# The table behind the table

*setmetatable attaches a second table of behaviour, consulted only when the first one comes up short.*

Luau Intermediate · Metatables · 9 min

---

A plain Luau table stores what you put in it and answers `nil` for everything else. A **metatable** is a second table you attach to the first, holding functions and values Luau consults when the ordinary rules run out.

One sentence carries the rest of this module: *the metatable is consulted on a miss, never on a hit*. If the key is already there, nothing you put in a metatable will ever run.

**`setmetatable` and `getmetatable`**

```luau
local defaults = { color = "grey", qty = 0 }

local lamp = setmetatable({ color = "red" }, { __index = defaults })

print(lamp.color, lamp.qty)         -- one hit, one miss
print(rawget(lamp, "qty"))          -- what is actually stored
print(getmetatable(lamp) ~= nil)
```

Prints `red  0`, then `nil`, then `true`. `lamp.color` is a hit and answers from the table itself; `lamp.qty` is a miss and falls through to `defaults`. `rawget` skips the metatable entirely and shows you the truth: nothing was ever stored under `qty`.

`setmetatable(t, mt)` attaches and returns `t`, which is why it is nearly always written as part of the expression that creates the table. `getmetatable(t)` hands it back. `rawget` and `rawset` do a read and a write with the metatable ignored.

Nothing here is magic and nothing is hidden: a metatable is an ordinary table whose keys happen to be names Luau looks for.

## What can be hooked

| Metamethod | Fires when | Used in this course for |
| --- | --- | --- |
| `__index` | You read a key the table does not have | Defaults, and the class pattern |
| `__newindex` | You write a key that is not already there | Read-only views, change tracking |
| `__tostring` | `tostring(t)`, which `print` calls for you | A slot that prints readably |
| `__eq` | `t1 == t2`, when both sides are tables sharing the metamethod | Value equality |
| `__len` | `#t` | A collection whose length is not its array part |
| `__call` | You call the table like a function | Callable objects |
| `__iter` | A generic `for` loops over the table | Iterators & Closures, the next module |

The arithmetic metamethods — `__add`, `__sub`, `__mul` and friends — exist too and are left to Luau Advanced; nothing in this course needs them.

**`__index` as a function**

```luau
local counts = setmetatable({}, {
	__index = function()
		return 0
	end,
})

counts.lamp += 1
counts.lamp += 1

print(counts.lamp, counts.rope)
```

Prints `2  0`. The first `+=` reads a missing key, gets `0` from the function and stores `1`; the second reads that stored `1` — a hit, so the function is not consulted — and stores `2`. `counts.rope` was never written, so it still answers `0`.

`__index` accepts either a **table** (look the key up in that instead) or a **function** (call it with the table and the key, use what it returns). A table is the right choice when the fallbacks are fixed; a function when they are computed.

The function form is what makes the counter above work with no setup at all, and the table form is what the class pattern two lessons from here is built on.

**Key points**

- A metatable is an ordinary table of behaviour attached with `setmetatable(t, mt)`, which returns `t`.
- It is consulted **on a miss, never on a hit** — a key already present never reaches it.
- `__index` may be a table (look there instead) or a function (call it with the table and the key).
- `rawget` and `rawset` do a read and a write with the metatable ignored.
- Every metamethod is named after the operation it intercepts: `__len` for `#`, `__call` for a call, `__iter` for a loop.
