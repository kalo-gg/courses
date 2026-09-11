# Writing an iterator

*The generic for calls a function until it returns nil, which is the whole protocol you must satisfy.*

Luau Intermediate · Iterators & Closures · 10 min

---

A generic `for` loop does something very simple: it calls a function over and over, binds what comes back to the loop variables, and stops as soon as the **first** returned value is `nil`.

That is the entire protocol. Anything that behaves that way can drive a loop, which means you can write one out of a closure in six lines.

**A range iterator, closure style**

```luau
local function range(n: number)
	local i = 0
	return function()
		i += 1
		if i <= n then
			return i
		end
		return nil
	end
end

for v in range(3) do
	print(v)
end
```

Prints `1`, `2`, `3`. `range` is called once and returns the closure; the loop then calls that closure four times — three values and a `nil` to stop.

Two things go wrong when people write their first one. The counter has to live **outside** the returned function, or it resets on every call. And the terminating return must be `nil` in the first position — returning `false`, or returning nothing at all when you meant to stop, ends the loop for different reasons or not at all.

Everything else is yours. The closure may return one value or several, may skip elements, may compute them.

## The three-value form

**Iterator, state, control**

```luau
local t = { "lamp", "rope" }

-- What pairs(t) really hands the loop: a function, a state and a control value.
local iterator, state, control = pairs(t)
print(type(iterator), state == t, control)

for k, v in iterator, state, control do
	print(k, v)
end
```

Prints `function  true  nil`, then `1  lamp` and `2  rope`. The loop calls `iterator(state, control)` each time and feeds the first value back in as the next control, which is how `next` walks a table without remembering anything itself.

So there are two ways to write an iterator. The **three-value** form keeps its position in the control value the loop hands back, so the iterator itself is stateless and can be shared — that is why `pairs` returns `next` rather than a closure per call.

The **closure** form keeps its position in a captured local. It allocates one function per loop, which is a real if small cost, and it is far easier to write. Write closures until a profiler tells you otherwise.

**Key points**

- A generic `for` calls a function until its **first** returned value is `nil`. That is the whole protocol.
- The closure form keeps its position in a captured local, declared outside the returned function.
- The three-value form — iterator, state, control — is stateless, which is why `pairs` uses it.
- `false` is a valid loop value; only `nil` in the first slot stops the loop.
- Write closures unless a profiler says the per-loop allocation matters.
