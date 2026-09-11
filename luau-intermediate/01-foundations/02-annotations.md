# Annotating values and functions

*The colon syntax on locals, parameters and returns, and the cast operator for the rest.*

Luau Intermediate · Types in Luau · 10 min

---

There is one piece of syntax to learn and it is a colon. It goes after a local's name, after a parameter's name, and after the closing parenthesis of a function to describe what comes back.

A function that returns two values gets both in parentheses. A value that may be missing gets a `?`, which the Typing Data module makes a whole lesson of.

**A local, a parameter, a return**

```luau
--!strict
local name: string = "lamp"
local qty: number = 2
local tag: string? = nil

local function restock(item: string, amount: number): (number, string)
	return amount, item .. " restocked"
end

local left, note = restock(name, qty)
print(name, qty, tag, left, note)
```

Prints `lamp  2  nil  2  lamp restocked` (tab separated). `left` and `note` need no annotation: the signature already says what they are.

## Casts, and when they are honest

Sometimes you know something the checker cannot work out — a value that came out of a generic container, a table you built in a loop. The cast operator `::` says "treat this expression as this type" and the checker takes your word for it.

That makes it useful and dangerous in the same breath. `value :: string` is a claim you can be wrong about. `value :: any` is worse: it is not a claim at all, it is switching the checker off for that value and for everything it flows into.

**A cast that helps, and one that hides**

```luau
--!strict
local raw: unknown = "12"

-- Honest: you know more than the checker, and you say exactly what.
local text = raw :: string
print(#text)

-- A lie: `any` is not a claim, it is the checker switched off.
local n = raw :: any
print(n + 1)
```

Prints `2`, then `13` — `"12" + 1` coerces at run time. The second cast let a string reach arithmetic with no complaint at all, which is exactly the bug the last lesson opened with.

The rule of thumb: a cast to a **specific** type is a note to the next reader that you checked. A cast to `any` is a note that you gave up, and it spreads — every value derived from an `any` is itself `any`.

When you genuinely do not know a type, `unknown` is the better vocabulary: it accepts anything going in and forces a narrowing before anything comes out.

**Key points**

- `name: Type` annotates a local or a parameter; `): Type` annotates the return, and two or more returns are parenthesised.
- `?` marks a value that may be `nil`.
- `::` asserts a type the checker could not prove — a claim you can be wrong about.
- `:: any` is not a claim, it is checking switched off, and it spreads to everything downstream.
- `unknown` is the honest "I do not know yet": it must be narrowed or cast before use.
