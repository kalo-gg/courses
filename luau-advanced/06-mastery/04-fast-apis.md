# An API that stays fast

*Return what exists, take an output parameter on hot paths, and say in the name when something mutates.*

Luau Advanced · Shipping Fast Luau · 9 min

---

Every optimization so far was something you did to your own code. This one is about the code you hand to other people: an implementation can only be as fast as its signature allows, and a getter that allocates makes every caller allocate whether they wanted to or not.

| Rule | Why |
| --- | --- |
| Return an existing value rather than a new table | The caller cannot un-allocate what you built for them |
| Take an output parameter on hot paths | It lets a caller supply one scratch object for a whole loop |
| Return several values instead of a table | Multiple returns are registers; a table is a heap object |
| Do not allocate in a getter | Getters get called in loops, by people who assume they are cheap |
| Say in the name when something mutates | `sorted` and `sortInPlace` are different promises |

**Two signatures for the same job**

```luau
-- Allocates a new table on every call.
local function addPoints(a: { x: number, y: number }, b: { x: number, y: number })
	return { x = a.x + b.x, y = a.y + b.y }
end

-- Writes into a table the caller owns; allocates nothing.
local function addPointsInto(out: { x: number, y: number }, a, b)
	out.x = a.x + b.x
	out.y = a.y + b.y
	return out
end

local a, b = { x = 1, y = 2 }, { x = 3, y = 4 }
local scratch = { x = 0, y = 0 }
print(addPoints(a, b).x, addPointsInto(scratch, a, b).y)
```

Prints `4	6`. Both are worth shipping: the first reads better at a call site used once, the second is the one a loop can afford.

## Fast and honest

There is a failure mode worth naming. An API that is fast **only if called in a particular order** — fill this before reading that, call `begin` before `step` — is a trap, because nothing about the signature says so and the penalty for getting it wrong is silent.

Two honest options. Document the contract precisely, next to the function rather than in a wiki. Or make it impossible to break: hand back a handle that only exists after `begin`, so the wrong order is not something a caller can write.

The same honesty applies to mutation. `sorted(list)` returning a new list and `sortInPlace(list)` reordering the caller's are both fine; a function called `sorted` that quietly does the second is not.

**Say it in the name**

```luau
local function sorted(list: { number }): { number }
	local copy = table.clone(list) -- one allocation, and the name says so
	table.sort(copy)
	return copy
end

local function sortInPlace(list: { number }): { number }
	table.sort(list) -- no allocation, and the name warns the caller
	return list
end

local values = { 3, 1, 2 }
print(table.concat(sorted(values), ","), table.concat(values, ","))
print(table.concat(sortInPlace(values), ","))
```

Prints `1,2,3	3,1,2`, then `1,2,3`. The middle column is the proof that `sorted` left the caller's list alone.

None of this means shipping only the fast form. `addPoints` and `addPointsInto` are both worth having: the first reads better at a call site used once, the second is the one a per-frame loop can afford. What is not worth having is a single function whose signature quietly forces the expensive shape on everybody.

**Key points**

- A signature decides what its callers can avoid allocating.
- Return existing values, offer an `out` parameter on hot paths, and prefer multiple returns to a table.
- Never allocate in a getter: getters are called in loops by people assuming they are cheap.
- An API that is only fast in a particular call order is a trap — document it precisely or make it unwritable.
