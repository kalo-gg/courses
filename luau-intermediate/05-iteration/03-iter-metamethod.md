# Looping an object directly

*Give a table an iter metamethod and the plain generic for works on it, with no helper at the call site.*

Luau Intermediate · Iterators & Closures · 10 min

---

Writing an iterator gives you `for v in bag:items() do`. That is fine, and it is one more thing every caller has to remember.

`__iter` removes it. A table whose metatable has an `__iter` can be looped directly — `for i, name in bag do` — because the loop asks the metatable how to iterate before it does anything else.

**A collection with `__iter`**

```luau
local Bag = {}
Bag.__index = Bag

function Bag.new(items: { string })
	return setmetatable({ items = items }, Bag)
end

function Bag.__iter(self)
	local i = 0
	return function()
		i += 1
		if self.items[i] ~= nil then
			return i, self.items[i]
		end
		return nil
	end
end

local bag = Bag.new({ "lamp", "rope" })
for i, name in bag do
	print(i, name)
end
```

Prints `1  lamp`, then `2  rope`. `__iter` is called once per loop with the object as `self`, and it returns exactly what the last lesson's iterators returned — so everything you already know applies.

Note that `__iter` returns a **fresh** closure every time it is called, which is why the same bag can be looped twice, and why nesting two loops over one bag works. An `__iter` that returned a shared iterator would work once and then quietly yield nothing.

`__iter` also supersedes `__pairs` and `__ipairs`, which exist in Lua 5.2 material and which Luau has said it will not add. If a tutorial reaches for those, it is not about Luau.

## Generalized iteration

The same loop form works on a plain table with no helper at all. `for k, v in t do` walks the array part in order and then the rest in an order you must not rely on — it is the one Luau extension to the loop that you will use every day.

So `pairs` is not needed for iteration any more. It is still worth knowing, because you will read it in every older file and because `next` is what it returns.

**`for k, v in t` on a plain table**

```luau
local t = { "lamp", "rope", tag = "tools" }

for k, v in t do
	print(k, v)
end
```

Prints `1  lamp`, then `2  rope`, then `tag  tools`. The array part comes first and in order; anything after it is in whatever order the table happens to store it, and a program that depends on that order is a program with a bug waiting.

> **`ipairs` and the table library ignore metamethods**
>
> This is a deliberate Luau decision and it catches people. `ipairs(proxy)` on a table whose values only exist through `__index` yields **nothing**, and `#proxy` is `0`; `table.insert`, `table.remove` and `table.concat` are the same. Only the generic `for` consults `__iter`. So a proxy table that looks complete through indexing will behave as empty the moment it meets the table library — keep the real data in a field and iterate that.

**Key points**

- `__iter` lets a generic `for` loop an object directly, with no helper call at the loop.
- It is a factory: build the iterator inside it so every loop starts fresh.
- `__iter` supersedes `__pairs` and `__ipairs`, which Luau will never add.
- `for k, v in t` works on a plain table too: array part in order, everything else in no order you may rely on.
- `ipairs`, `#` and the table library ignore metamethods entirely — a proxy table looks empty to all of them.
