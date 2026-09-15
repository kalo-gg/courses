# What it costs on the wire

*One byte for the event, then exactly what your types said, and nothing else.*

Blink · Best Practices & Capstone · 10 min

---

Every `Write` function the generator emits opens with a comment: `-- Allocate BLOCK: N bytes`. N is the fixed part of the packet, one byte for the event's id and then exactly what your types declared. Variable-length fields, strings, buffers and arrays, allocate again after the block, so N is the floor rather than the whole cost.

**net.blink**

```blink
event Snapshot {
    From: Server,
    Type: Unreliable,
    Call: SingleSync,
    Data: (u8, vector, u16)
}
```

**Server.luau (generated)**

```luau
function SerdesFunctions.WriteEVENT_Snapshot(Value1: number, Value2: Vector3, Value3: number): ()
	-- Allocate BLOCK: 16 bytes
	local BLOCK_START = Allocate(16)
	buffer.writeu8(SendBuffer, BLOCK_START + 0, 0)
	buffer.writeu8(SendBuffer, BLOCK_START + 1, Value1)
	local Vector = Value2
	buffer.writef32(SendBuffer, BLOCK_START + 2, Vector.X)
	buffer.writef32(SendBuffer, BLOCK_START + 6, Vector.Y)
	buffer.writef32(SendBuffer, BLOCK_START + 10, Vector.Z)
	buffer.writeu16(SendBuffer, BLOCK_START + 14, Value3)
end
```

Offset 0 is the event's id. Everything after it is your declaration, in order, with no framing, no field names and no padding.

| Piece | Bytes |
| --- | --- |
| The event itself (its id) | 1 |
| `u8` / `i8` / `boolean` | 1 |
| `u16` / `i16` / `f16` | 2 |
| `u32` / `i32` / `f32` | 4 |
| `f64` | 8 |
| `vector` | 12 (three `f32`), or 3 x the width of `vector<T>` |
| `string` / `buffer` / array | 1 or 2 for the length, then the contents |
| `Instance` | 0 in the buffer: it rides the RemoteEvent's instance array |
| A unit enum | 1 |
| A tagged enum | 1 for the tag, then the variant's fields |

The sandbox on this page reads that same comment and prints `block: <Name> = N bytes` for every event it compiles. That is the number to budget against, and it is checkable long before anything ships: change a type, press Run, read the new figure.

> **`Instance` is free in bytes and expensive in risk**
>
> It costs nothing in the buffer. But a non-optional `Instance` that arrives as `nil` raises a deserialisation error and the rest of the packet is dropped: streaming, a client-only instance and a despawn between send and receive all cause it. Write `Instance(Player)?` unless you can prove it exists on both sides.

## Working a budget

The working, spelled out: 1 byte of id, 1 for the `u8`, 12 for the vector and 2 for the `u16`. Sixteen, times 20, times 40. Then compare it against what the same three values cost as a Luau table through a RemoteEvent, which is the gap Blink's own benchmark measured as 41.81 Kbps against 559,364.

**Key points**

- `-- Allocate BLOCK: N bytes` is the compiler's own answer, and the sandbox prints it as `block:`.
- Every packet carries one id byte, then exactly the fields you declared.
- A bare `vector` is twelve bytes; `vector<i16>` is six and is usually enough.
- Strings, buffers and arrays cost a length prefix plus contents, so bound them.
- `Instance` costs nothing in the buffer, and a non-optional one can drop the whole packet.
