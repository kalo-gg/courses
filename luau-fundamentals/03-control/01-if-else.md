# Making a decision

*if, elseif and else, the comparison operators, and why one chain beats a staircase of blocks.*

Luau Fundamentals · Control Flow · 9 min

---

An `if` runs a block only when a condition is truthy — and "truthy" means exactly what the last lesson said: anything but `false` and `nil`.

The shape is `if … then … end`, with `elseif` for as many extra questions as you like and one optional `else` at the bottom. There are no braces and no colon: `then` opens the block and `end` closes it.

**if, elseif, else**

```luau
local score = 88

if score >= 90 then
	print("grade S")
elseif score >= 80 then
	print("grade A")
elseif score >= 70 then
	print("grade B")
else
	print("grade F")
end
```

Prints `grade A`. The branches are tested top to bottom and **the first true one wins** — the `score >= 70` line is never even asked, because `score >= 80` already answered.

The comparison operators are `==`, `~=`, `<`, `<=`, `>` and `>=`. Two of them are worth pointing at:

- **`~=` is "not equal".** Luau has no `!=`; typing one is a syntax error.
- **`==` compares tables by identity, not contents.** Two separately built tables with the same fields are not equal. Numbers and strings compare by value, as you would expect.

All six always produce a real boolean, so `if a == b` needs no `== true` after it.

## One chain, not a staircase

**The same decision, flattened**

```luau
-- A staircase: every else opens a whole new block
if score >= 90 then
	print("S")
else
	if score >= 80 then
		print("A")
	else
		print("F")
	end
end

-- The same decision, one level deep
if score >= 90 then
	print("S")
elseif score >= 80 then
	print("A")
else
	print("F")
end
```

Identical behaviour. `elseif` exists precisely so that a chain of questions stays flat: the second version has one `end`, no indentation creep, and reads as the list of cases it is.

Conditions can be combined with `and`, `or` and `not`, and because those short-circuit you can put the cheap or protective test first: `if room and room.locked then` never evaluates `room.locked` when `room` is `nil`. That ordering is a real technique, not a style preference, and Module 4 leans on it heavily.

Luau also has an `if … then … else` **expression**, which produces a value rather than choosing a statement: `local label = if won then "win" else "loss"`. It always needs the `else`, because an expression must always produce something.

**Key points**

- `if … then … end`, with `elseif` and one optional `else`; the first true branch wins and the rest are skipped.
- `~=` is "not equal"; `!=` does not exist in Luau.
- `==` compares tables by identity, so two look-alike tables are not equal.
- Chain with `elseif` instead of nesting; `and`/`or` short-circuit, so guard on the left.
