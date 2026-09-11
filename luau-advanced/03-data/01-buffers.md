# Raw bytes

*A buffer is a fixed block of bytes with no keys and no iteration: you decide what the layout means.*

Luau Advanced · Buffers & Vectors · 10 min

---

A thousand particles as a thousand tables is a thousand heap objects, each with a hash part, each visited by the collector, each reached through a pointer chase. When every one of them holds the same six numbers, that is a lot of machinery for very little information.

A **buffer** is the alternative: a fixed-size block of raw bytes you read and write by byte offset. One object, whatever the record count.

**create, len, fromstring, tostring**

```luau
local b = buffer.create(16) -- 16 zeroed bytes; the size never changes
print(buffer.len(b))

local hi = buffer.fromstring("hi")
print(buffer.len(hi), buffer.tostring(hi))
```

Prints `16`, then `2	hi`. `buffer.len` is the only way to ask how big one is — `#b` does not work on a buffer.

A buffer is deliberately unlike a table. It has **no keys**, so nothing is named. It is **not iterable**, so there is no `for … in`. It has **no metatable**, so nothing can be hooked. And its size is fixed at creation: there is no appending.

What you get in exchange is that the bytes are exactly the bytes you wrote, in the order you wrote them, in one contiguous allocation. The layout is not described anywhere — it lives in the code that reads and writes it, which is why a stride constant and a pair of helper functions are not optional extras but the whole design.

## When a buffer wins

| Situation | Buffer | Table |
| --- | --- | --- |
| Many records of the same fixed shape | One allocation for all of them | One heap object each |
| Fields are all numbers | Packed to the exact widths | Every value boxed the same way |
| Read and written in a hot loop | Offset arithmetic, no hashing | A hash or array lookup per field |
| Sent over the wire or saved | Already a byte layout | Needs serializing first |
| Fields added and removed as you design | Every offset shifts | Add the key and move on |
| Sparse, or keyed by name | The wrong tool entirely | What tables are for |

The last two rows are the honest half: a buffer trades flexibility for density, and most data wants the flexibility.

**fill and copy**

```luau
local b = buffer.create(4)
buffer.fill(b, 0, 255) -- every byte from offset 0 to the end
print(buffer.readu8(b, 0), buffer.readu8(b, 3))

local dst = buffer.create(4)
buffer.copy(dst, 0, b, 0, 2) -- two bytes only
print(buffer.readu8(dst, 0), buffer.readu8(dst, 2))
```

Prints `255	255`, then `255	0`: the copy moved two bytes, so byte 2 of the destination is still the zero it was created with.

`fill` and `copy` are the two bulk operations, and both are byte-level: they know nothing about your records. `copy` is how you grow a store, since a buffer cannot be resized — create a bigger one and copy the old bytes into it, which is exactly what a table's array part does for you invisibly.

**Key points**

- A buffer is a fixed-size block of raw bytes: no keys, no iteration, no metatable, no growing.
- `buffer.len` is the only way to read its size; `#` is not defined on buffers.
- It wins for many records of one fixed numeric shape, and loses for anything sparse or named.
- The layout lives in your read and write helpers, so a stride constant is part of the design, not a detail.
