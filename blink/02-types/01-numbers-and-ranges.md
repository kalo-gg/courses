# Numbers and ranges

*Nine number types, and a range that shrinks one of them costs you nothing.*

Blink · The Type System · 10 min

---

Blink's number types are named after what they are: a prefix saying unsigned, signed or floating point, and a bit width. `u8` is an unsigned eight-bit integer, `i16` a signed sixteen-bit one, `f32` a single-precision float. The width is not a hint. It is the number of bits that field occupies in the packet, every time it is sent.

| Type | Bytes | Min | Max |
| --- | --- | --- | --- |
| `u8` `u16` `u32` | 1 2 4 | 0 | 255 · 65,535 · 4,294,967,295 |
| `i8` `i16` `i32` | 1 2 4 | −128 · −32,768 · −2,147,483,648 | 127 · 32,767 · 2,147,483,647 |
| `f16` `f32` `f64` | 2 4 8 | −65,504 · −16,777,216 · −2^53 | 65,504 · 16,777,216 · 2^53 |

From `src/Settings.luau`, which is what the compiler enforces. The docs page gives `f16` as ±2048; that is the largest *integer* an f16 represents exactly, a different claim, and the compiler accepts ±65,504.

**numbers.blink**

```blink
type Health = u8(0..100)
type UserId = f64
type Score = u16(0..)
type Offset = i8(..0)
```

A range in brackets after the type. `f64` is the one that holds a Roblox UserId without losing digits.

There are four range forms. Full, `0..100`, bounds both ends. Half-open, `0..` or `..100`, bounds one and leaves the other at the type's own limit. And an exact form, a bare `100`, which accepts that value and nothing else. A range must sit inside the type's bounds, so `u8(0..300)` is refused before anything is generated.

## What a range actually buys

**Client.luau (generated, read-only)**

```luau
function SerdesFunctions.ReadEVENT_Bounded(): (number)
	-- Read BLOCK: 1 bytes
	local BLOCK_START = Read(1)
	local Value = buffer.readu8(RecieveBuffer, BLOCK_START + 0)
	if Value < 0 then error(`Expected "Value" to be larger than 0, got {Value} instead.`) end
	if Value > 100 then error(`Expected "Value" to be smaller than 100, got {Value} instead.`) end
	return Value
end
```

The same event without the range emits the identical function minus the two `if` lines.

So a range does not add a field and it does not change the block size of a number. It adds a `<` and a `>` check where the data arrives, which is the cheapest validation you will ever write and the only one that cannot be forgotten. For the variable-length types it does more than that, and the next lesson is about exactly that.

> **Pick the type from the domain, not from Luau**
>
> Luau has one number type, so the habit is to reach for the widest thing and stop thinking. Ask instead what the value *is*. A percentage is a `u8`. A round counter is a `u16`. A world position component is an `f32`, because a float you can see is a float you can round. `f64` is for identifiers and timestamps, where the digits matter.

## Your turn

**Key points**

- Nine number types, named prefix plus bit width, and the width is the cost on the wire.
- Ranges come in four forms: full, half-open either way, and an exact value.
- A range on a number adds two comparisons where the data is read and changes no sizes.
- A range outside the type's own bounds is `[E3009]`, caught at compile time.
