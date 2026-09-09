# One step, three phases

*A single step runs the intervals, flushes the deferred nodes, then advances the animators.*

Fluid · Animation & Timing · 11 min

---

In a game, Fluid runs itself: on load it spawns a thread bound to `RunService.PreRender` and steps the graph every frame. `provide_scheduler` takes that job back.

**Taking over**

```luau
local step = fluid.provide_scheduler()

-- Fluid's own loop is canceled; nothing advances until you say so
step(1 / 60)
```

It cancels the internal thread and returns the step function it was calling.

That step does three things, in this order, and knowing the order explains a lot about when values settle — why an animation never lags a frame behind the state it follows, and why a list rebuild lands before the animators run.

## Deferred nodes: the middle phase

`fluid.deferred(fn, seed?)` is an `effect` that waits. Instead of running the moment a dependency changes, its node is queued and evaluated when the scheduler flushes — so several writes in one frame cost **one** run rather than one each. It is the only thing in the public API that puts a node in that queue directly.

**Four writes, one run**

```luau
local destroy = fluid.root(function()
	local n = fluid.source(0)
	local eager, batched = 0, 0

	fluid.effect(function() n(); eager += 1 end)
	fluid.deferred(function() n(); batched += 1 end)

	n(1) n(2) n(3)
	print("before the step:", eager, batched) --> 4  1
	step(1)
	print("after the step: ", eager, batched) --> 4  2
end)

destroy()
```

Both run once when created. The three writes then cost the effect three runs and the deferred node one.

> **`async` nodes are not in the deferred queue**
>
> It is a natural assumption — the source calls async evaluations "deferred" — but the queue only ever receives nodes created with `evaluation == "deferred"`, and an async node's mode is `"async"`, which the graph groups with **eager**. An async node is invalidated the moment its dependencies change, not on the next flush. What is deferred about it is the callback, which runs on its own thread.

> **Two different `step`s, one name**
>
> The function Fluid hands back takes a **delta time in seconds**: `step(1 / 60)` is one frame. Kalo's sandbox wraps it in a helper that takes a **frame count** instead, so `step(60)` in every lesson before this one means "sixty frames at 1/60s each" — one second. Same name, inverse units. In your own game you are calling Fluid's version, so pass a delta time; everything you have run in this course went through the same path, just clocked by hand.

**Key points**

- `fluid.provide_scheduler()` cancels Fluid's `RunService.PreRender` loop and hands you its step function.
- That function takes a **delta time in seconds**; the sandbox's `step(frames)` takes a frame count. Same name, inverse units.
- One step: intervals, then the deferred flush, then `lerp`, then `spring`.
- `fluid.deferred` is an effect that waits for that flush, so several writes in a frame cost one run.
- `async` nodes are **not** in that queue — they are invalidated eagerly; only their callback is deferred.
