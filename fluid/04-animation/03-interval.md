# Work on a clock

*Repeating work at a rate in hertz, with one free call before the clock starts counting.*

Fluid · Animation & Timing · 10 min

---

`interval` runs a callback repeatedly, driven by Fluid's scheduler rather than a loop you write. Its full signature is `interval(func: (dt: number) -> T, hz: number?, offset: number?) -> Readable<T>` — four things worth reading off it, none of which are obvious from a two-argument call.

**Hertz, not seconds — and one free call**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local ticks = 0

	local latest = fluid.interval(function(dt)
		ticks += 1
		return ticks * 10
	end, 2) -- twice per second

	print(ticks, latest())  --> 1  10   (the immediate call, before any stepping)

	step(60)                -- one second
	print(ticks, latest())  --> 2  20
end)

destroy()
```

One second at 2 Hz is **one** gated call, not two: the gate fires at frame 31 and again at frame 62. The count reaches 2 because of the free call at creation.

> **The easy mistake**
>
> Most timing APIs take a period, so `1/30` looks like the natural way to say "every frame". Here it means the opposite. If your interval seems never to fire, check whether you have passed a period where a frequency was wanted.

## It hands you a value, not just a callback

`interval` returns a **readable** holding whatever the callback last returned — the README calls it a "utility to poll values". That is the feature: an interval polling a `Humanoid`'s health, a server's player count or a datastore's cached value gives you a reactive value like any other, and bindings follow it.

**Polling into a binding**

```luau
local destroy = fluid.root(function()
	local ticks = 0

	local latest = fluid.interval(function(dt)
		ticks += 1
		return ticks * 10
	end, 2)

	local label = fluid.create("TextLabel")({
		Text = function()
			return "value: " .. latest()
		end,
	})

	print(label.Text) --> value: 10   (the free call at creation)
	step(60)
	print(label.Text) --> value: 20
end)

destroy()
```

The interval's readable is a dependency like any other, so the `Text` binding follows it without a loop or a connection anywhere: `value: 10` from the free call at creation, then `value: 20` after the second's one gated call at 2 Hz.

One behavior worth knowing before you debug it: if the callback errors, Fluid warns once and the interval goes **quiet** — later failures are swallowed until a call succeeds, at which point warnings resume. So an interval that logged once and then went silent has not recovered; it has been failing all along.

> **Count the immediate call when you do the arithmetic**
>
> The gate is `if elapsed <= 1 / hz then return end`, so a gated call needs *strictly more* than the period — one frame more, in practice. At 4 Hz that is frames 16, 32 and 48, with the fourth landing at 64: three gated calls inside the first second, plus the free one, so you count four.

**Key points**

- `fluid.interval(fn, hz, offset?)` measures a **rate in hertz**, not a period. `interval(fn, 30)` is thirty times a second.
- The callback runs once immediately at creation with `dt = 0`, before any stepping.
- It returns a **readable** holding the callback's latest return value, so an interval can feed a binding.
- `offset` shifts the first gated call, for staggering intervals that would otherwise collide.
- The callback may not yield. Because it runs once at creation, a failure on *that* call is raised where you created the interval; a later failure warns once and then goes quiet, silently, until a call succeeds.
