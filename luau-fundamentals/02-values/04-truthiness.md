# True, false and nil

*Only false and nil are falsy, which makes zero and the empty string true and explains and or.*

Luau Fundamentals · Numbers, Strings & Booleans · 9 min

---

The whole rule fits in one sentence: **everything is truthy except `false` and `nil`.**

That is shorter than most languages' answer, and it is the part people get wrong. `0` is truthy. `""` is truthy. The string `"false"` is truthy. An empty table is truthy. If you are coming from JavaScript or Python, half of what you know about falsy values does not apply here.

**Everything except two things**

```luau
print(0 and "truthy" or "falsy")       -- truthy
print("" and "truthy" or "falsy")      -- truthy
print("false" and "truthy" or "falsy") -- truthy
print(nil and "truthy" or "falsy")     -- falsy
print(false and "truthy" or "falsy")   -- falsy
```

No `if` here — branching is the next module. `a and b or c` is the expression form of the same idea, and reading it is the point of the next two steps.

## and, or, not

`and` and `or` do not return booleans. They return **one of their operands**.

- `a and b` gives `a` when `a` is falsy, and `b` otherwise. It is asking "did the first one fail? then stop here".
- `a or b` gives `a` when `a` is truthy, and `b` otherwise. It is asking "was the first one any good? if not, take the second".
- `not a` is the odd one out: it always returns a real `true` or `false`.

Both short-circuit, so the right-hand side is not evaluated at all when the left one already decided the answer.

**The default-value idiom**

```luau
local given = nil
local shown = given or "stranger"
print("welcome, " .. shown) -- welcome, stranger

-- The trap: it also replaces a legitimate false.
local muted = false
local setting = muted or true
print("setting is", setting) -- true, which is not what muted said
```

`x or default` is the standard way to fill in a missing value, and it is exactly wrong when `false` is a value you meant. That is the one case where you have to ask `x == nil` instead.

> **`x or default` swallows a real `false`**
>
> If `x` is legitimately `false` — a muted flag, a "has the player opted out" setting, a disabled toggle — then `x or default` silently substitutes the default and your setting never sticks. The fix is to test for absence explicitly rather than for falsiness: ask whether `x == nil`. This trap comes back in Module 6 with a name, and it is worth recognising now.

One more consequence: comparisons like `==`, `<` and `~=` always produce a real boolean, so `if score > 10` and `if (score > 10) == true` mean the same thing and the second one is noise. Write the comparison and stop.

**Key points**

- Only `false` and `nil` are falsy. `0`, `""` and `"false"` are all truthy.
- `and` and `or` return an operand, not a boolean, and short-circuit.
- `x or default` is the default-value idiom, and it silently eats a legitimate `false`.
- Test `x == nil` when absence and `false` mean different things.
