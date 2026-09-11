# Values in both directions

*yield is a call that returns when someone resumes you, and that one sentence explains the plumbing.*

Luau Advanced · Coroutines · 9 min

---

The part of coroutines people re-derive every time is which value lands where. There is one sentence that settles it:

**`yield` is a call that returns when someone resumes you.**

Its arguments go out to the resumer, as the extra results of their `resume`. Their next `resume`'s arguments come back in, as the results of that same `yield`. Read it as an ordinary call whose return is merely delayed and the plumbing stops being mysterious.

**The arguments to resume become the results of yield**

```luau
local co = coroutine.create(function(a: number, b: number)
	print("args", a, b) -- from the first resume
	local c = coroutine.yield(a + b) -- a + b goes out; c comes back
	print("resumed with", c)
	return c * 10
end)

print(coroutine.resume(co, 1, 2))
print(coroutine.resume(co, 5))
```

Prints `args	1	2`, `true	3`, `resumed with	5`, `true	50`. Four lines, and every value in them crossed the boundary in one of the two directions.

## The first resume is different

That exception is worth stating plainly, because it is where the confusion starts. When you create a coroutine, its function has not begun. There is no `yield` waiting to return, so the **first** `resume`'s arguments go to the function's **parameters**, exactly like an ordinary call.

Every resume after that has a suspended `yield` to hand its arguments to. A generator that takes no setup often ignores this entirely; a coroutine that is configured once and then driven repeatedly uses both mechanisms in the same run, and it is worth being able to see which is which.

**A coroutine that accumulates what it is given**

```luau
local co = coroutine.create(function()
	local total = 0
	while true do
		local n = coroutine.yield(total) -- hands out the total, waits for more
		if n == nil then
			break
		end
		total += n
	end
	return total
end)

coroutine.resume(co) -- run as far as the first yield
print(select(2, coroutine.resume(co, 5)))
print(select(2, coroutine.resume(co, 7)))
```

Prints `5`, then `12`. `select(2, …)` drops the survival flag so only the yielded value is shown.

That coroutine is a small protocol: it hands out its state, waits, folds in whatever it is given, and hands out the new state. Written as a function you would need somewhere to keep `total` between calls; written as a coroutine, the local *is* the state, and it survives because the function never finished.

**Key points**

- `yield` is a call that returns when someone resumes you — everything else follows from that.
- `yield`'s arguments become the resumer's extra results; the next resume's arguments become `yield`'s results.
- The first resume is the exception: its arguments go to the function's parameters.
- The final resume's extra results are whatever the function returned.
