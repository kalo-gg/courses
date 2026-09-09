# What reactivity costs

*What each kind of node costs, and where the deferred phase does the saving for you.*

Fluid · Mastery & Migration · 11 min

---

Fluid is fast enough that most screens never need this lesson. The ones that do are the ones with a per-frame source — a health bar following `Humanoid.Health`, a leaderboard following a replicated table — where a single write fans out into a hundred property assignments, sixty times a second.

| Node | When it runs | What it costs |
| --- | --- | --- |
| `source` | Never — it holds a value | A version bump and an invalidation walk over its dependents |
| `derive` | On read, if stale | The callback, once, plus a coroutine (dev mode is always on) |
| `effect` | Immediately, on every change | The callback per change, plus a coroutine |
| `deferred` | Once per flush in which it was invalidated | The callback once per frame, however many writes there were |
| A property binding | Immediately, on every change | The callback plus one property assignment |
| A reactive child list | Once per flush | A rebuild of the list's diff |
| `async` | Invalidated eagerly; callback on its own thread | A thread per evaluation |

## Where the work actually is

Two costs are structural. Every reactive evaluation runs inside a coroutine, because that is how Fluid detects a yield — and dev mode is not optional, so you pay for it in production too. And every write walks the dependents to mark them stale, so a source with a hundred readers costs a hundred marks whether or not anything recomputes.

**One derive, one wake-up**

```luau
local destroy = fluid.root(function()
	local health = fluid.source(100)
	local bucket = fluid.derive(function()
		return math.floor(health() / 10) -- 0..10 instead of 0..100
	end)

	local runs = 0
	fluid.effect(function()
		bucket()
		runs += 1
	end)

	for hp = 100, 91, -1 do
		health(hp)
	end
	print("writes: 10, downstream runs:", runs) --> 2
end)

destroy()
```

Ten writes, one bucket change, two runs of the effect — one at creation and one when the bucket actually moved.

> **A table source is always a change**
>
> Fluid refuses to compare two mutable tables, so writing a rebuilt list into a source invalidates the whole list node even when nothing in it moved. A leaderboard rebuilt every frame therefore re-diffs every frame. Keep the entry tables and rewrite only the array around them.

**Same tables, no rebuild**

```luau
local ada, grace = { name = "Ada" }, { name = "Grace" }

local destroy = fluid.root(function()
	local players = fluid.source({ ada, grace })
	local builds = 0

	local rows = fluid.for_values(players, function(entry, key, active)
		builds += 1
		return entry.name
	end)

	rows()
	players({ ada, grace }) -- the same two tables, a new array
	rows()
	players({ ada, grace })
	rows()

	print("builds:", builds) --> 2
end)

destroy()
```

Two writes after the first read and the transform never runs again — the array changed, the entries did not. Build the entry tables inline instead (`players({ { name = "Ada" }, { name = "Grace" } })`) and the same script prints **6**: every write is a full re-diff, because `for_values` keys its per-entry state by the value and two mutable tables are never equal.

Three levers, in the order to try them: collapse the value with a `derive` so most writes stop early; move the expensive reader behind a `fluid.deferred` so a frame's writes cost one run; and keep table identities stable so a list diff has nothing to do.

**Key points**

- `source` and `derive` are cheap; anything eager is what costs per write.
- Dev mode is always on: every reactive evaluation runs in a coroutine, in production too.
- A node whose value did not change wakes nothing — so a derive that collapses values is the cheapest optimization available.
- `fluid.deferred` turns many writes in a frame into one run.
- Two mutable tables are never equal, so a rebuilt table source always re-diffs the list.
