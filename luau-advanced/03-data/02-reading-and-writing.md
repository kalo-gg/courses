# Offsets and strides

*Index times stride plus field offset is the whole technique, and an out-of-range read errors rather than lying.*

Luau Advanced · Buffers & Vectors · 10 min

---

Every buffer access names a **byte offset**. The read and write functions come in families by width — one byte, two, four, eight — and the width you read with has to be the width you wrote with, because nothing records which one you used.

**writeu8, writef32, writestring and their reads**

```luau
local b = buffer.create(16)

buffer.writeu8(b, 0, 255) -- one byte at offset 0
buffer.writef32(b, 4, 1.5) -- four bytes at offset 4
buffer.writestring(b, 8, "abc") -- three bytes at offset 8

print(buffer.readu8(b, 0))
print(buffer.readf32(b, 4))
print(buffer.readstring(b, 8, 3))
```

Prints `255`, `1.5`, `abc`. `readstring` needs the count because a buffer stores no length prefix and no terminator.

An **offset** is a position in bytes. A **stride** is the size of one whole record. Put the two together and the entire technique is one line of arithmetic:

`index * stride + field offset`

Six 32-bit floats make a 24-byte stride, with position at offsets 0, 4 and 8 and velocity at 12, 16 and 20. Particle 3's velocity y is therefore at `3 * 24 + 16`, which is byte 88.

**A stride of 24 bytes: six f32s**

```luau
local STRIDE = 24 -- x, y, z, vx, vy, vz, four bytes each
local store = buffer.create(2 * STRIDE)

local function setPos(i: number, x: number, y: number, z: number)
	local base = i * STRIDE
	buffer.writef32(store, base + 0, x)
	buffer.writef32(store, base + 4, y)
	buffer.writef32(store, base + 8, z)
end

setPos(1, 3, 4, 5)
print(buffer.readf32(store, STRIDE + 4)) -- record 1, field y
```

Prints `4`. `base` is computed once per call, and every field is a constant offset from it — which is why the helper exists rather than the arithmetic being repeated at each call site.

## Getting it wrong

Two mistakes are possible and they behave very differently.

Reading **past the end** raises `buffer access out of bounds`. That is genuinely good news: it fails loudly at the access rather than handing back plausible garbage that surfaces three systems later.

Reading with the **wrong width** does not error at all. Read the bytes of an `f32` with `readu32` and you get a perfectly valid, completely meaningless number. Nothing can catch this for you, which is the strongest argument for writing `set` and `get` helpers and never touching offsets anywhere else.

**Key points**

- Every buffer access is a byte offset; `index * stride + field offset` is the whole technique.
- The width you read with must be the width you wrote with — nothing records which you used.
- Reading past the end raises `buffer access out of bounds` rather than returning garbage.
- Keep the offsets inside `set` and `get` helpers so no call site can get them wrong.
