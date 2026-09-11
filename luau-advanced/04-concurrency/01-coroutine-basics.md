# A function that pauses

*create, resume, yield and status: four calls that turn a function into something you can stop halfway.*

Luau Advanced · Coroutines · 10 min

---

Every function you have written so far runs to completion. A **coroutine** does not have to. It can stop in the middle, hand a value back to whoever started it, and be started again later from exactly where it stopped — with its locals, its loop counters and its position intact.

That single capability is what every `wait`, every task scheduler and every generator is built out of. There is nothing else underneath.

**create, resume, yield, status**

```luau
local co = coroutine.create(function(x: number)
	local y = coroutine.yield(x + 1) -- stops here, hands back x + 1
	return y * 2 -- resumes here, with y from the next resume
end)

print(coroutine.resume(co, 1))
print(coroutine.resume(co, 10))
print(coroutine.status(co))
```

Prints `true	2`, then `true	20`, then `dead`. The first result of `resume` is always the survival flag; the rest is what the coroutine yielded or returned.

A coroutine is in one of four states. **suspended** is where it starts and where every `yield` leaves it. **running** is the one it is in while its own code executes. **normal** means it resumed another coroutine and is waiting for that one. **dead** means it returned or errored, and cannot be resumed again.

The other thing to notice is that `resume` **never throws**. An error inside the coroutine comes back as `false` plus the message, and so does resuming a dead one. That is a deliberate design: the caller decides what an inner failure means.

## Not a thread

Luau calls the value a `thread`, and the word is misleading. Coroutines are **cooperative**: exactly one of them runs at a time, and it keeps running until it chooses to yield. Nothing is preempted, nothing runs in parallel, and there are no data races to reason about.

That is the trade. You never need a lock, and you can also hang everything by writing a loop that never yields.

**Watching the state change**

```luau
local co = coroutine.create(function()
	print("inside", coroutine.status(coroutine.running()))
	coroutine.yield()
end)

print("before", coroutine.status(co))
coroutine.resume(co)
print("after yield", coroutine.status(co))
coroutine.resume(co)
print("finished", coroutine.status(co))
```

Prints `before	suspended`, `inside	running`, `after yield	suspended`, `finished	dead`. `coroutine.running()` is how a coroutine gets a handle on itself.

**Key points**

- A coroutine stops mid-function and resumes with its locals and position intact.
- Four states: suspended, running, normal, dead — and dead is final.
- `resume` never throws: its first result says whether the coroutine survived.
- Coroutines are cooperative, so nothing is preempted and nothing runs in parallel.
