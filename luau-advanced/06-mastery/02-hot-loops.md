# The shape of a hot loop

*Hoist the lookups, allocate nothing, flatten the data, and leave the cold code alone.*

Luau Advanced · Shipping Fast Luau · 10 min

---

A hot loop is one whose body runs often enough that the per-iteration cost is the program's cost. Everything worth doing to one comes down to moving work **out** of the body — and the list is short enough to hold in your head.

| Move | What it removes | From Module |
| --- | --- | --- |
| Hoist your own lookups | A field-chain read per iteration on your tables — not on `math`, which is resolved at load | How Luau Runs |
| Allocate nothing in the body | A heap object per iteration | The cost you cannot see |
| Avoid varargs in the body | Argument packing on every call | How Luau Runs |
| Avoid metamethod dispatch | A function call per access | Intercepting everything |
| Index a buffer instead of chasing tables | A pointer chase per field | Buffers & Vectors |
| Keep number types stable | The one thing codegen needs | Native code generation |

Every row is something already covered. This lesson is the assembly, not new material.

**The same update, twice**

```luau
local config = { offset = { dx = 1 } }

-- Reads config.offset.dx on every iteration: two field lookups, each time.
local function shiftSlow(list: { { x: number } })
	for _, p in list do
		p.x = math.floor(p.x + config.offset.dx)
	end
end

-- The same chain read once, before the loop. math.floor is left alone on
-- purpose: it is a library chain, resolved when the script loads, so
-- hoisting it would change nothing.
local function shiftFast(list: { { x: number } })
	local dx = config.offset.dx
	for _, p in list do
		p.x = math.floor(p.x + dx)
	end
end

local points = { { x = 1.4 }, { x = 2.6 } }
shiftSlow(points)
shiftFast(points)
print(points[1].x, points[2].x)
```

Prints `3	4`. Both compute the same thing; the second reads `config.offset.dx` once, instead of paying two lookups on every element. What it does *not* hoist is `math.floor` — the one move in this lesson that a Lua 5.1 programmer would make and Luau makes pointless.

## What not to do

The rules above have a precondition that is easy to skip: the loop has to actually be hot. Applied to code that runs once at startup, every one of them makes the program harder to read and no faster.

That cost is real. `math.floor(p.x + dx)` needs the reader to go and find what `dx` is and where it was read; a buffer offset needs them to know the stride; a hoisted scratch table needs them to check that nothing holds a reference to it after the iteration. Each is worth paying where the loop matters and worth refusing where it does not.

**The same loop, unrolled by hand**

```luau
local values = { 1, 2, 3, 4 }

-- Faster? Nothing says so. Certainly harder to change.
local total = 0
total += values[1]
total += values[2]
total += values[3]
total += values[4]

print(total, #values)
```

Prints `10	4`. It also breaks the moment the list holds five things — and the compiler already unrolls small loops at `--!optimize 2` when it judges that worthwhile.

Hand-unrolling is the clearest case of the trade going the wrong way. It hard-codes a length into the code, so the loop and the data can drift apart silently — and at `--!optimize 2` the compiler already unrolls small loops where it judges that worthwhile, which is a judgement it can revisit on every build and you cannot.

**Key points**

- Hoist your own lookups (not library chains like `math.floor`, which Luau resolves at load), allocate nothing, avoid varargs and metamethod dispatch, flatten the data, keep types stable.
- A numeric `for` evaluates its limit once; hoisting `#list` out of one is folklore.
- Every one of these costs readability, so it has to be paid for by a loop that is genuinely hot.
- Measure first and measure again — the last step is knowing when to stop.
