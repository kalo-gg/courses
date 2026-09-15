# Strings, buffers and arrays

*Three variable-length types, and the bound you give them picks their length prefix.*

Blink · The Type System · 10 min

---

A number's width is known from its name, so the reader knows where the next field starts. A string's is not, so the length has to travel with it. Blink writes that length as an integer in front of the data, and it picks the smallest integer that can express the **maximum** you declared. Declare no maximum and it has to assume the largest one.

**Client.luau (generated, read-only)**

```luau
function SerdesFunctions.WriteEVENT_Free(Value: string): ()
	-- Allocate BLOCK: 3 bytes
	local BLOCK_START = Allocate(3)
	buffer.writeu8(SendBuffer, BLOCK_START + 0, 0)
	local Length = #Value
	buffer.writeu16(SendBuffer, BLOCK_START + 1, Length)
	Allocate(Length)
	buffer.writestring(SendBuffer, SendOffset, Value, Length)
end
```

`Data: string`. The block is 3 bytes: one event id and a `u16` length.

**Client.luau (generated, read-only)**

```luau
function SerdesFunctions.WriteEVENT_Short(Value: string): ()
	-- Allocate BLOCK: 2 bytes
	local BLOCK_START = Allocate(2)
	buffer.writeu8(SendBuffer, BLOCK_START + 0, 1)
	local Length = #Value
	buffer.writeu8(SendBuffer, BLOCK_START + 1, Length)
	Allocate(Length)
	buffer.writestring(SendBuffer, SendOffset, Value, Length)
end
```

`Data: string(..255)`. The block is 2 bytes: the same event id and a `u8` length. The `0` and `1` in each first `writeu8` are the event ids.

The two functions differ by one byte per message, and the only difference in the source was `string` against `string(..255)`. On a chat event that is nothing. On a movement event at sixty hertz it is 60 bytes a second per player, and it cost you a pair of brackets.

| Type | Written | Unbounded prefix | Bounded to 255 or less |
| --- | --- | --- | --- |
| string | `string(3..20)` | `u16` | `u8` |
| buffer | `buffer(..900)` | `u16` | `u8` |
| array | `f64[1..50]` | `u16` count | `u8` count |

A string's range bounds its length in characters, an array's bounds its element count.

## Arrays

An array is any type followed by `[]`, with an optional range *inside* the brackets bounding the element count: `u8[]`, `u8[..8]`, `f64[1..50]`. The count itself can never exceed 65,535, the `ARRAY_BOUNDS` limit in `src/Parser.luau`, because the prefix is at most a `u16`. Each element is then written at its own type's width, so a bounded count and a wide element type is still a wide packet.

> **`buffer` is the escape hatch**
>
> If you already have a packer, a `buffer` field carries your bytes through untouched and still gets Blink's batching, its length prefix and its bound check. It is the right answer for a voxel chunk or a compressed replay frame, and the wrong answer for three numbers you did not feel like naming.

## Your turn

**Key points**

- Variable-length data carries its length on the wire, and the declared maximum picks the prefix type.
- A maximum of 255 or less gives a `u8` prefix; anything else, including no bound at all, gives a `u16`.
- An array is `type[]`, with the element count bounded inside the brackets and capped at 65,535.
- `buffer` carries bytes you packed yourself and still gets the prefix, the bound check and the batching.
