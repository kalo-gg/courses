# Wire types

*How a remote's payload travels: widths, buffers and tables, and the checks every packet passes on arrival.*

Alloy · Alloy on Roblox · 10 min

---

A shot remote fires every time a player clicks: an aim point and a power. Sent as plain numbers, that is three 64-bit doubles, and the server cannot trust any of them.

Alloy reads a **wire layout** from the parameter types. It packs the payload into a `buffer` when that is smaller, and checks every packet when it arrives.

## Widths pack a buffer

```alloy
struct Shot
  @f32 x: number
  @f32 y: number
  @u8 power: number
end

remote Fire(shot: Shot) from client -- a buffer: 4 + 4 + 1 bytes
remote Chat(message: string) from client -- a table
```

A width attribute sets a number's size on the wire. Without one, a number takes 64 bits, so this `Shot` would pack into 24 bytes, not 9.

**ship (lines 7 and 8)**

```luau
local Fire = __alloy.remote({ name = "Fire", kind = "event", from_client = true, from_server = false, params = { "shot" }, defaults = {  }, attrs = {}, wire = { { fields = { { "x", "f32" }, { "y", "f32" }, { "power", "u8" } }, struct = Shot } } }) -- a buffer: 4 + 4 + 1 bytes
local Chat = __alloy.remote({ name = "Chat", kind = "event", from_client = true, from_server = false, params = { "message" }, defaults = {  }, attrs = {}, wire = { "str" }, pack = false }) return nil -- a table
```

The layout is written into the declaration's line. `pack = false` marks a remote the compiler sends as a table: a lone string gains nothing in a buffer.

Structs, record types and arrays pack field by field, and a struct comes back through its constructor, metatable and all. A unit enum crosses as its name and a payload enum beside the buffer, and the reader checks the variant. A `Player`, any Instance and datatypes such as `Vector3` travel beside the buffer as the engine sends them. A map is not packed.

The receiver trusts nothing. Before the handler runs, each value is checked against its type and width, each Instance against its class, and each length against the bytes in the packet. A packet that does not fit is dropped, and the handler never sees it. `@ratelimit` counts a packet before it is read, so a flood costs no decoding.

| Attribute | Effect |
| --- | --- |
| `@u8`, `@u16`, `@u32` | Unsigned whole numbers in 1, 2 or 4 bytes |
| `@i8`, `@i16`, `@i32` | Signed whole numbers in 1, 2 or 4 bytes |
| `@f32` | A 4-byte float, instead of the default 8 bytes |
| `@wire(buffer)` | Always pack into a buffer. A map or an untyped table is then a `WireType` error. |
| `@wire(table)` | Always send a table, still checked against the layout on arrival |
| `@unreliable` | Roblox drops a payload over 900 bytes, so an unbounded string, array or map is a `WireType` error |

**Key points**

- The compiler picks a buffer or a table for each remote, whichever sends fewer bytes. `@wire` forces one.
- `@u8` to `@f32` set a number's width. Without one, a number takes 64 bits.
- A value too big for its width raises at the fire, naming the path to the value.
- The receiver checks every packet against the layout and drops one that does not fit.
