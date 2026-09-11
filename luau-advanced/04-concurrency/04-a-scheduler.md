# Writing a scheduler

*A queue of coroutines resumed in turn is the whole idea behind every task library you have used.*

Luau Advanced · Coroutines · 11 min

---

Put the last three lessons together and you get a **scheduler**: a list of coroutines and a `step` that resumes each of them once. That really is all a task library is. Roblox's adds real time and an engine's frame loop to the same shape; the shape is below.

**A queue of coroutines, stepped in turn**

```luau
local queue: { thread } = {}

local function spawn(fn: () -> ())
	table.insert(queue, coroutine.create(fn))
end

local function step()
	local alive: { thread } = {}
	for _, co in queue do
		coroutine.resume(co) -- never throws, so one bad task cannot stop the loop
		if coroutine.status(co) ~= "dead" then
			table.insert(alive, co)
		end
	end
	queue = alive -- dead tasks are dropped
end
```

Twelve lines. `resume` reporting instead of raising is what lets `step` keep going when a task fails.

**Driving it**

```luau
spawn(function()
	for i = 1, 2 do
		print("a", i)
		coroutine.yield()
	end
end)

spawn(function()
	print("b")
end)

step()
print("alive", #queue)
step()
print("alive", #queue)
```

Prints `a	1`, `b`, `alive	1`, `a	2`, `alive	1`. Task b finished in the first step and was dropped; task a yielded and stayed.

## Cancellation

A task that should stop before it finishes is **closed**, not simply forgotten. `coroutine.close` releases it and marks it dead, so anything holding a reference sees a dead coroutine rather than a suspended one that will silently never run again.

After closing, it must not be resumed. The scheduler above already handles this by accident — `resume` on a dead coroutine returns `false` rather than raising — but relying on that is how a cancelled task quietly becomes a task nobody notices is gone. Check the status, or drop the reference.

**Cancelling one task**

```luau
local victim = coroutine.create(function()
	coroutine.yield()
	print("this never runs")
end)

coroutine.resume(victim) -- runs to the yield
print(coroutine.close(victim), coroutine.status(victim))
print(coroutine.resume(victim))
```

Prints `true	dead`, then `false` with `cannot resume dead coroutine`. The `print` inside the coroutine never happens, which is the point of closing rather than abandoning.

One thing this scheduler is not is fair. It resumes in queue order, every live task, every step, with no priorities and no budget — so a task that does a great deal of work between yields delays everything behind it. That is inherent to cooperative scheduling, and it is why a real task library gives you both a way to yield often and a way to say how urgent something is.

**Key points**

- A scheduler is a list of coroutines and a `step` that resumes each one once.
- `resume` reporting instead of raising is what lets one failing task not stop the loop.
- Rebuild the queue rather than mutating the one you are iterating.
- Cancel with `coroutine.close`, and never resume a task after closing it.
