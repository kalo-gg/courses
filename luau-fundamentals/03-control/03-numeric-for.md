# Counting loops

*for i = start, stop, step counts for you, and the loop variable is fresh on every pass.*

Luau Fundamentals · Control Flow · 10 min

---

When you know how many times something should happen, a `while` makes you write the counter, the increment and the condition by hand — three places to get it wrong. A **numeric `for`** writes all three for you.

`for i = 1, 5 do` gives you `i` as 1, 2, 3, 4 and 5. Both bounds are **inclusive**, which is the first thing to unlearn if you come from a language where the end is exclusive.

**for i = 1, 5**

```luau
for i = 1, 5 do
	print(i)
end

-- The same thing as a while, written out longhand
local i = 1
while i <= 5 do
	print(i)
	i += 1
end
```

Both print 1 through 5. The `for` version has nowhere to put an off-by-one: the counter, the test and the step are one line and the language owns all three.

A third number after the stop is the **step**, and it may be negative. `for i = 10, 0, -5` counts 10, 5, 0. A step of `3` from 1 to 10 gives 1, 4, 7, 10 and then stops, because 13 is past the end.

All three numbers are evaluated **once**, before the first pass. Changing the variable you computed the stop from, inside the body, does not change how many times the loop runs.

**Counting down, and by threes**

```luau
for i = 10, 0, -5 do
	print("down", i)
end

for i = 1, 10, 3 do
	print("by threes", i)
end
```

The first prints 10, 5, 0. The second prints 1, 4, 7, 10 — the step keeps going until it would pass the stop value, and never overshoots it.

## The variable is new each pass

`i` is scoped to the loop: it does not exist before the `for` and it is gone after the `end`. More than that, **each pass gets its own fresh `i`** rather than reusing one variable.

Most of the time this makes no difference at all. It matters the moment a function created inside the loop captures `i` — each function then remembers its own value instead of all of them sharing the last one. That is the payoff, and Functions & Scope is where it is collected.

Assigning to `i` inside the body is legal and affects only that pass, because the next pass gets a new one. It is also confusing, so do not.

**Key points**

- `for i = start, stop, step` counts for you, and **both bounds are inclusive**.
- The step may be negative; leave it out and it is `+1`.
- Start, stop and step are evaluated once, before the loop, so the body cannot move the goalposts.
- `i` is scoped to the loop and is fresh on every pass.
