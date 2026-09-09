# Effects

*Run something when state changes, know what you subscribed to, and never yield inside one.*

Fluid · The Reactive Graph · 12 min

---

A `derive` produces a value. An `effect` produces a *consequence*: it runs its callback now, and again whenever a source it read changes. Anything that reaches outside the graph — a print, a remote, a sound — goes in one.

**Reacting to a change**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local health = fluid.source(100)

	fluid.effect(function()
		print("health is now", health())
	end)
	--> health is now 100   (it runs once immediately)

	health(80)
	--> health is now 80
end)

destroy()
```

Reading `health()` inside the effect is what subscribes it. Nothing registers the dependency by hand.

> **Effects need a stable scope too**
>
> Like `derive`, an `effect` outside a `root` fails with `cannot call effect without a parent scope`. An effect with no owner could never be stopped, so Fluid will not create one.

## An effect remembers its own last result

Both `effect` and `derive` take the same callback type, `(previous: T) -> T`: the callback is handed whatever it returned last time, and its return becomes the next `previous`. `effect`'s optional second argument seeds the first run. It is the tidy way to compare against what you last saw without a variable outside the scope.

**Diffing against the previous value**

```luau
local destroy = fluid.root(function()
	local health = fluid.source(100)

	fluid.effect(function(previous)
		local current = health()
		if current < previous then
			print("took", previous - current, "damage")
		end
		return current
	end, 100)

	health(80) --> took 20 damage
end)

destroy()
```

`100` is the seed. Without it the first run would compare against `nil`.

## No reactive callback may yield

Fluid runs every effect, derive, reactive property and `interval` callback inside a coroutine, and checks afterwards whether it is still suspended. If it is, the thread is canceled and the run is abandoned half-finished. That rules out `task.wait`, `:WaitForChild`, `:InvokeServer` and `:GetAsync` everywhere in the graph.

| Where | May it yield? | What happens if it does |
| --- | --- | --- |
| `effect`, `derive`, `deferred` | No | First run: a hard error where you created it. Later runs: a warning, the run abandoned, a stale value left behind |
| A function property in `create` / `bind` | No | Same as above — it is an ordinary reactive node |
| An `interval` callback | No | It runs once at creation, and a yield or error *there* is raised where you created the interval (`logging.error`). Later: one warning, then the interval goes quiet — further failures are silent until a call succeeds |
| An `async` callback | **Yes** | Nothing. It is the exception, and the reason `async` exists |

> **The second failure is the one you will meet**
>
> Yield on the *first* run and you get a hard error at the point of creation, which is easy. Yield on a *later* re-run and Fluid only logs a warning: the callback is canceled half-way, your game carries on with a stale value, and there is no stack trace at the call site. The check is on unconditionally — `dev_mode` is hardcoded `true` in the shipped source, whatever the README says about `_G.__DEV__`.

> **You cannot reproduce that one here**
>
> Kalo's sandbox has no real task scheduler, so its `task.wait` returns immediately rather than yielding — an effect that calls it runs clean in the playground and would fail in a game. `coroutine.yield()` does trip the check if you want to see it.

**Key points**

- `fluid.effect(fn)` runs `fn` now and again whenever a source it read changes. Side effects live here.
- Subscribing happens per read: what the callback calls is what it depends on.
- `effect(fn, seed)` hands the callback its own previous return value, so you can diff without an outside variable.
- No reactive callback may yield. First run: an error. Later runs: a warning and a stale value.
- `fluid.async` is the one node whose callback is allowed to wait.
