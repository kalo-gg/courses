# Types that compute types

*type function runs at check time, and keyof and index derive a type from another one.*

Luau Advanced · The Type System, Deeply · 11 min

---

Every type so far was one you wrote out. Luau can also **compute** one. A `type function` is a function that runs during type checking, receives types as arguments and returns a type; the built-in ones — `keyof`, `index` — are the same idea packaged.

Nothing here exists at run time. A type function is executed by the checker and erased before the VM sees the file, which is why the examples below print ordinary values.

**A type function, in full**

```luau
--!strict
-- Runs at check time. `t` arrives as a value of the checker's `types`
-- library, and whatever this returns is used as the type.
type function same(t)
	return t
end

type Point = { x: number, y: number }
type Same = same<Point>

local p: Same = { x = 1, y = 2 }
print(p.x, p.y)
```

Prints `1	2`. `same` is deliberately trivial; a real one inspects `t` with the `types` library and builds something new from it.

## The built-in ones

| Written as | Computes | Typical use |
| --- | --- | --- |
| `keyof<T>` | A union of `T`'s keys | A parameter that must name a field of `T` |
| `index<T, K>` | The type stored at key `K` of `T` | The return type of a lookup into `T` |
| `rawget<T, K>` | The same, ignoring `T`'s metatable | Reaching past an `__index` chain |
| `setmetatable<T, M>` | `T` with `M` attached | Describing what `setmetatable` produced |
| `getmetatable<T>` | `T`'s metatable type | Reading a class's shared table back out |

`keyof` is the workhorse. Given a table type it produces the union of that table's keys, so a function taking `keyof<Stats>` accepts `"hp"` and `"mp"` and rejects everything else — without you writing the union out and without it drifting when `Stats` gains a field.

One footnote worth knowing: as of 0.737 `keyof` orders the union it produces lexicographically. That is invisible until an error message prints the type, and then it explains why the order is not the one you declared.

> **This is the newest corner of the language**
>
> User-defined `type function` is far younger than the rest of Luau's type system. The RFCs in the `luau-lang/rfcs` repository are the reference; the docs site lags them, and behaviour has moved between minor versions. Everything shown here parses and runs in the 0.733 sandbox — but write against the RFC, and expect to revisit it.

**keyof and index typing a lookup**

```luau
--!strict
type Colors = { red: string, blue: string }
type ColorName = keyof<Colors> -- "blue" | "red"
type ColorValue = index<Colors, "red"> -- string

local palette: Colors = { red = "#f00", blue = "#00f" }

local function get(name: ColorName): ColorValue
	return palette[name]
end

print(get("red"), get("blue"))
```

Prints `#f00	#00f`. Add a key to `Colors` and `get` accepts it immediately; misspell one at a call site and the checker catches it.

It is worth being precise about *when* each of these happens, because the two worlds are easy to blur once a type starts looking like a computation. A type function's body executes exactly once per check, in the checker's process, and never again; `keyof` expands there too. By the time the VM has the bytecode, all of it is gone.

**Key points**

- A `type function` runs during type checking and returns a type; it is erased before the VM runs.
- `keyof<T>` is the union of `T`'s keys, `index<T, K>` the type stored at `K`.
- A derived type cannot drift out of step with the type it came from.
- This is the newest part of the language: the RFCs are the reference, and behaviour may still move.
