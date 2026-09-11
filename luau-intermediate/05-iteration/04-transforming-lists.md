# map, filter and reduce, by hand

*Luau ships no map or filter, so every codebase writes them; here is the version worth copying.*

Luau Intermediate · Iterators & Closures · 10 min

---

There is no `table.map` and there is not going to be one. Every Luau codebase grows its own, usually three times in three files, and the versions that survive are the ones that were typed properly the first time.

This is where Generics & Functions pays off. `map<T, U>(list: { T }, f: (T) -> U): { U }` says everything: it takes a list of one thing and a function that turns one of those into another, and gives back a list of the other.

**`map` and `filter`, generic and typed**

```luau
--!strict
local function map<T, U>(list: { T }, f: (T) -> U): { U }
	local out: { U } = {}
	for i, v in list do
		out[i] = f(v)
	end
	return out
end

local function filter<T>(list: { T }, keep: (T) -> boolean): { T }
	local out: { T } = {}
	for _, v in list do
		if keep(v) then
			table.insert(out, v)
		end
	end
	return out
end

local names = { "lamp", "rope", "pin" }
print(table.concat(map(names, string.upper), ","))
print(table.concat(filter(names, function(s) return #s == 4 end), ","))
```

Prints `LAMP,ROPE,PIN`, then `lamp,rope`. `map` keeps the index so the result lines up with the input; `filter` uses `table.insert` because its result is shorter and must stay contiguous.

## reduce, and why it is last

**`reduce` with a seed**

```luau
--!strict
local function reduce<T, A>(list: { T }, seed: A, step: (A, T) -> A): A
	local acc = seed
	for _, v in list do
		acc = step(acc, v)
	end
	return acc
end

local qtys = { 2, 1, 3 }
print(reduce(qtys, 0, function(a, n) return a + n end))
print(reduce({ "a", "b" }, "", function(a, s) return a .. s end))
```

Prints `6`, then `ab`. Two type parameters, because the accumulator need not be the element type — that is exactly what makes `reduce` able to express both of these.

`reduce` comes last because it is the one people reach for too early. A sum written as a `for` loop is clearer than a sum written as a reduce, and it stays clearer until the accumulator is doing something a loop would have made ugly.

The honest rule: use `map` when the shape stays the same and the values change, `filter` when the values stay the same and the shape shrinks, and a plain loop when you are not sure.

One habit that makes all three safe: **never mutate the input**. Build a new table and return it. A `filter` that removes elements from the list it was handed will surprise the caller, and it will surprise the loop it is running inside even more.

The versions above allocate a table each. That is the right default; if a profiler ever says otherwise, the fix is a hand-written loop at that one call site, not a cleverer helper.

**Key points**

- Luau ships no `map`, `filter` or `reduce`; every codebase writes them, so write them typed.
- `map<T, U>(list: { T }, f: (T) -> U): { U }` — the two parameters are what make it reusable.
- `filter` appends with `table.insert` so the result has no holes; `map` keeps the index.
- `reduce<T, A>` takes a seed, and its accumulator type need not match the element type.
- Never mutate the input list — build a new one and return it.
