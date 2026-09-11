# Returning more than one value

*Luau functions return any number of values, and where you put the call decides how many survive.*

Luau Fundamentals · Functions & Scope · 10 min

---

A Luau function can return as many values as it likes, and the caller can take as many as it wants. This is not a trick with tables — there is no wrapper object anywhere — and it is used constantly in the standard library. You met it in Module 2 when `string.gsub` printed a count you did not ask for.

**Two returns, two locals**

```luau
local function bounds(a, b)
	if a < b then
		return a, b
	end
	return b, a
end

local low, high = bounds(9, 4)
print(low, high) -- 4  9

-- Take fewer than were offered and the rest are discarded.
local onlyLow = bounds(9, 4)
print(onlyLow) -- 4

-- Ask for more than were offered and you get nil.
local a, b, c = bounds(9, 4)
print(a, b, c) -- 4  9  nil
```

`return a, b` and `local low, high = …` are the two halves of the same idea. Both sides are flexible, and neither ever errors about the count.

## Truncation is positional

Here is the rule, and it is the whole lesson: **a call in the last position of a list contributes all of its values; anywhere else it contributes exactly one.**

"A list" means an argument list, the right-hand side of an assignment, a `return`, or a table constructor. So `print(two())` shows both values, and `print(two(), "end")` shows only the first — the call is no longer last.

Wrapping a call in an extra pair of parentheses forces it down to one value wherever it sits. `(two())` is one value, always.

**The same call, four placements**

```luau
local function two()
	return 1, 2
end

print(two())         -- 1  2   : last in the list, everything survives
print(two(), "end")  -- 1  end : not last, truncated to one
print((two()))       -- 1       : parentheses force exactly one
print(two(), two())  -- 1  1  2 : first truncated, second expands

local t = { two(), two() }
print(#t)            -- 3
```

The fourth line is the whole rule in one statement: the same function, called twice in one argument list, contributing one value and then two. The table on the last line ends up holding `1`, `1`, `2`.

> **This is why `string.gsub` surprised you**
>
> Back in Finding text, `print(string.gsub(s, " ", "-"))` printed a mysterious number after the string. Now you know why: `gsub` returns two values, the call was in the last position of `print`'s argument list, so both came through. `(string.gsub(…))` — the extra parentheses — is the fix, and it is the same fix for every two-value function in the library.

One consequence worth internalising: a multi-value call in the middle of a **table constructor** contributes one element, and at the end contributes all of them. `{ f(), g() }` has one element from `f` and as many as `g` returns. If you want them all regardless of position, `table.pack(f())` collects them — the next lesson's subject.

**Key points**

- A function returns any number of values and the caller takes any number; neither side errors.
- A call in the **last** position of a list contributes every value; anywhere else, exactly one.
- Extra parentheses — `(f())` — force a single value wherever the call sits.
- Missing values arrive as `nil`, so `local a, b, c = two()` leaves `c` nil.
