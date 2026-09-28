# Vector3

*One type for points, offsets and directions, and the operations that answer how far and which way.*

Roblox Fundamentals · Space · 10 min

---

A coin should only be collected by a player within 20 studs of it. A cannon should fire only at targets in front of it. Both are questions about where things are, and in Roblox the answer is a `Vector3`: three numbers, `X`, `Y` and `Z`, in studs.

The same type means different things depending on how you use it. `part.Position` is a **position**, a point in the world. `b - a` is an **offset**, how to get from `a` to `b`. An offset with its length set to 1 is a **direction**.

## Distance and direction

```luau
local coin = workspace.Coin.Position
local player = Vector3.new(10, 3, -4)

local offset = coin - player      -- from the player to the coin
local distance = offset.Magnitude -- how far, in studs
local direction = offset.Unit     -- which way, with length 1

if distance <= 20 then
	print("close enough to collect")
end
```

End minus start: `coin - player` points **from** the player **to** the coin. Swap them and the offset points the other way, with the same `Magnitude`.

A `Vector3` cannot be changed in place: `part.Position.Y = 5` errors with `Y cannot be assigned to`. Build a new vector instead, as in `part.Position += Vector3.yAxis * 5`.

`Vector3.zero` is `(0, 0, 0)`, `Vector3.one` is `(1, 1, 1)` and `Vector3.yAxis` is `(0, 1, 0)`, world up.

| Expression | Gives | Use it for |
| --- | --- | --- |
| `a + b`, `a - b` | Vector3 | A point moved by an offset; the offset between two points |
| `v * 3`, `v / 2` | Vector3 | Scaling: `direction * 5` is 5 studs that way |
| `v.Magnitude` | number | Length: `(a - b).Magnitude` is the distance |
| `v.Unit` | Vector3 | The direction alone, with length 1 |
| `a:Dot(b)` | number | For unit vectors: 1 the same way, 0 at right angles, -1 opposite |
| `a:Lerp(b, t)` | Vector3 | The point `t` of the way from `a` to `b`; 0.5 is the midpoint |

> **The zero vector has no direction**
>
> `Vector3.zero.Unit` does not error. Its components are NaN (not a number), and NaN spreads through every sum it touches. When two points can coincide, check `offset.Magnitude > 0` before you take `Unit`.

## Is it in front?

`Dot` answers the cannon's question. Take the way the cannon faces and the direction to the target, both with length 1. Their dot product is positive when the target is in front, 0 when it is exactly to the side and negative when it is behind. The next lesson shows where a part's facing comes from; here it is a plain vector.

```luau
local facing = Vector3.new(0, 0, -1) -- this cannon faces -Z
local toTarget = (target - cannon).Unit

if facing:Dot(toTarget) > 0 then
	print("in front")
end

local halfway = cannon:Lerp(target, 0.5) -- the midpoint
```

The dot product of two unit vectors is the cosine of the angle between them, so `> 0.7` means within about 45° of straight ahead.

**Key points**

- `b - a` is the offset from `a` to `b`; its `Magnitude` is the distance and its `Unit` the direction.
- Vectors cannot be changed in place: build a new one rather than setting `.Y`.
- For unit vectors, `Dot` is positive in front, 0 to the side and negative behind.
- The `Unit` of a zero vector has NaN components, so check the `Magnitude` first.
