# wrap, and what happens to errors

*wrap hides the thread behind a plain function and lets errors through; create keeps both in your hands.*

Luau Advanced · Coroutines · 10 min

---

`coroutine.create` plus `coroutine.resume` is the explicit form: you hold the thread, you check the flag, you ask for the status. `coroutine.wrap` is the same machinery with the thread hidden — it hands back a plain function, and calling it resumes.

**coroutine.wrap gives a plain function**

```luau
local nextValue = coroutine.wrap(function()
	coroutine.yield(1)
	coroutine.yield(2)
	coroutine.yield(3)
end)

print(nextValue(), nextValue(), nextValue())
```

Prints `1	2	3`. No survival flag, no `select`, no thread variable — which is exactly why generators are almost always written this way.

|  | `create` + `resume` | `wrap` |
| --- | --- | --- |
| What you hold | A `thread` | A function |
| On success | `true` plus the yielded values | Just the yielded values |
| On an error inside | `false` plus the message | The error is raised into the caller |
| `status` and `close` | Available | No thread to pass them |
| Reads best for | A scheduler that must survive a failing task | A generator, or anything you want to read like a call |

Not a beginner form and an expert form: they answer different questions about who owns failure.

**Where the error goes**

```luau
local bad = coroutine.wrap(function()
	error("boom")
end)
local ok, err = pcall(bad) -- wrap raises, so the caller needs a pcall
print(ok, string.match(tostring(err), "boom"))

local co = coroutine.create(function()
	error("boom")
end)
print(coroutine.resume(co)) -- resume reports instead of raising
```

Prints `false	boom`, then `false` with the message. Same failure, two places it surfaces.

`coroutine.close` is the other half of the story. It releases a suspended coroutine and marks it dead without running anything more inside it — the way a scheduler cancels a task. Closing an already-dead coroutine is harmless and returns `true`; resuming a closed one gives the same `false, "cannot resume dead coroutine"` as any other dead coroutine.

## Yielding across a boundary

There is one place a `yield` cannot happen: inside a metamethod. Yielding from `__index`, `__newindex` or any other hook raises `attempt to yield across metamethod/C-call boundary`.

This is deliberate rather than an oversight — a metamethod is called from inside the VM's own machinery, which has no way to be suspended and resumed. It is also the error people hit when a lazy-loading `__index` reaches for something that waits, and the fix is always the same shape: make the waiting explicit at the call site instead of hiding it behind an access.

**Key points**

- `wrap` hands back a plain function and lets errors propagate; `create` plus `resume` reports them as values.
- A wrapped coroutine has no thread, so `status` and `close` are unavailable for it.
- `close` cancels a suspended coroutine; closing a dead one is harmless, resuming one is not possible.
- A metamethod cannot yield: `attempt to yield across metamethod/C-call boundary` is by design.
