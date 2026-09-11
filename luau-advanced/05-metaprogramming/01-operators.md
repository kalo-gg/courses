# Making a type do arithmetic

*The arithmetic and comparison hooks, which operand's metatable runs, and Luau's own rule for __eq.*

Luau Advanced · Metamethods in Full · 10 min

---

You already build classes with `__index` and print them with `__tostring`. The set is much larger than that, and the rest of it is what turns a type into something you can *compute with* rather than only call methods on.

Every entry below is called when an operator meets a value the VM cannot handle itself. None of them is magic: each is a plain function stored under a reserved key in a metatable.

| Metamethod | Operator | Metamethod | Operator |
| --- | --- | --- | --- |
| `__add` | `a + b` | `__unm` | `-a` |
| `__sub` | `a - b` | `__concat` | `a .. b` |
| `__mul` | `a * b` | `__eq` | `a == b` |
| `__div` | `a / b` | `__lt` | `a < b` |
| `__idiv` | `a // b` | `__le` | `a <= b` |
| `__mod` | `a % b` | `__pow` | `a ^ b` |

`>` and `>=` have no metamethods of their own: Luau derives them from `__lt` and `__le` with the operands swapped.

**A Vec2 with __add, __unm and __tostring**

```luau
local Vec2 = {}
Vec2.__index = Vec2

function Vec2.new(x: number, y: number)
	return setmetatable({ x = x, y = y }, Vec2)
end

function Vec2.__add(a, b)
	return Vec2.new(a.x + b.x, a.y + b.y)
end

function Vec2.__unm(a)
	return Vec2.new(-a.x, -a.y)
end

function Vec2.__tostring(v)
	return "(" .. v.x .. ", " .. v.y .. ")"
end

local sum = Vec2.new(1, 2) + Vec2.new(3, 4)
print(tostring(sum), tostring(-sum))
```

Prints `(4, 6)	(-4, -6)`. The metatable is the class table itself, which is the usual arrangement: `__index` for methods, the operator hooks beside them.

## Which operand's metamethod runs

For a binary operator Luau tries the **left** operand's metatable first, and only if it has no handler does it try the right one's. That is what makes `Vec2 * 2` and `2 * Vec2` able to share one `__mul` — write the handler to cope with either order and both expressions reach it.

`__eq` is the exception, and Luau's rule differs from Lua's in two ways worth knowing. It **is** called even when the two operands are the same object, where Lua short-circuits on identity. And it is **not** called when the two operands carry different `__eq` handlers — that comparison falls back to raw identity rather than picking one of them.

**__eq, twice**

```luau
local function tagged(name: string)
	return setmetatable({}, {
		__eq = function()
			print(name .. "'s __eq ran")
			return true
		end,
	})
end

local A = tagged("A")
local B = tagged("B")

print(A == A) -- Luau calls __eq even for the same object
print(A == B) -- different handlers, so neither runs
```

Prints `A's __eq ran`, `true`, then `false`. The second comparison never reaches a handler at all.

Both of those rules cost something. Because `__eq` runs for identical objects, a handler that walks a big structure is paid for even by `x == x`, so an identity fast path at the top of the handler is usually worth writing. And because differing handlers fall back to identity, two types that both think they know how to compare themselves will silently disagree — which is a reason to keep one shared metatable per type rather than building a fresh one per value.

**Key points**

- Every arithmetic and comparison operator has its own metamethod, with no fallbacks between them.
- `>` and `>=` are the exception: Luau derives them from `__lt` and `__le`.
- The left operand's metatable is tried first, which is how one handler serves both operand orders.
- Luau calls `__eq` even for identical objects, and skips it entirely when the two operands carry different handlers.
