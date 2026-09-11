# Three numbers, one value

*vector is a VM primitive with its own library, read-only components, and no heap cost at all.*

Luau Advanced · Buffers & Vectors · 9 min

---

A position is three numbers that always travel together. Written as a table that is a heap object per position; written as three locals it is three things to keep in step. Luau's answer is **`vector`**: three numbers the VM treats as a single value, with arithmetic operators and a small library.

The important word is *value*. A vector lives in registers like a number does. Passing one, returning one, adding two — none of it allocates.

**create, dot, cross, magnitude, and arithmetic**

```luau
local x = vector.create(1, 0, 0)
local y = vector.create(0, 1, 0)

print(vector.dot(x, y))
print(vector.magnitude(vector.create(3, 4, 0)))
print(vector.cross(x, y))
print(x + y)
print(typeof(x))
```

Prints `0`, `5`, `0, 0, 1`, `1, 1, 0`, `vector`. Printing a vector shows its three components separated by `, `, so it never looks like a table address.

`typeof` reports `vector` because this is a VM primitive, not a library type built on tables. Components read as `v.x`, `v.y`, `v.z` and are **read-only** — assigning to one raises `attempt to index vector with 'x'`. To change a vector you build a new one, which costs nothing.

One clarification worth making early: this is **core Luau**, not Roblox. Roblox's `Vector3` is a separate engine datatype with a different library, and `vector.create` does not produce one.

## Where it pays

The win is not only the syntax. A table of three numbers is a heap object: allocated, collected, and reached through a pointer. A vector is three floats sitting in registers. In a loop over ten thousand positions that is the difference between ten thousand allocations per frame and none — which is exactly what makes the final project able to report zero heap growth over sixty frames.

The cost is that a vector is always three components and always numbers. Anything with a fourth field, a name or a string belongs in a table.

**Normalizing and scaling**

```luau
local v = vector.create(0, 3, 4)
local unit = vector.normalize(v)

print(unit)
print(vector.magnitude(unit))
print(unit * 10)
```

Prints `0, 0.6000000238418579, 0.800000011920929`, then `1`, then `0, 6, 8`. Components are single-precision, which is why the middle number is not a clean `0.6` — the magnitude still comes back as exactly `1`.

**Key points**

- `vector` is a VM primitive: `typeof` reports `vector`, and it never touches the heap.
- Components read as `.x`, `.y`, `.z` and are read-only; build a new vector instead of writing one.
- The arithmetic operators are the API — `pos + vel * dt` is one allocation-free expression.
- This is core Luau; Roblox's `Vector3` is a separate engine type.
