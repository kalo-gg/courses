# Strict mode and debugging

*Let Vide double-check your reactive code, and recognize its common error messages.*

Vide · Shipping a Vide App · 10 min

---

`vide.strict = true` turns on a set of checks that catch reactive mistakes while they are cheap to fix. It is on automatically when Vide loads at optimization level 0 or 1 (Studio) and off at O2 (production builds), so you rarely set it by hand outside the sandbox.

`vide.strict` is a flag on the library table, read each time a scope runs, so set it before any scope exists: at the top of the entry script, or in a test harness. Reading it back tells you which build you are in. The Kalo sandbox starts with it off so printed output is easy to follow; every exercise below turns it on explicitly.

| Check | What it catches |
| --- | --- |
| Reactive scopes run **twice** per update | Effects and derived sources with side effects that are not idempotent, and cleanups that do not fully undo their work |
| Yields inside reactive scopes error | `task.wait`, `WaitForChild` and other yields in effects, which would leave the graph half-updated |
| Duplicate values in `values()` | Two rows that would share one scope |
| Duplicate properties at one nesting depth | `Text` set in two sibling nested tables, e.g. `{ { Text = "a" }, { Text = "b" } }` |
| Destroying an active scope | Calling a destructor from inside the scope it destroys |
| Better error traces | Stack traces that point at your binding instead of Vide internals |

**The double run in action**

```luau
vide.strict = true

local count = source(0)
local ran = 0

root(function()
	effect(function()
		count()
		ran += 1
	end)
end)

print(ran) -- 2
count(1)
print(ran) -- 4
```

If your effect prints or increments something, strict mode makes that visible immediately. Pure effects are unaffected.

## Errors you will meet

| Message | Cause | Fix |
| --- | --- | --- |
| `cannot use effect() outside a stable or reactive scope` | Calling `effect`, `derive` or a component with effects outside `root`/`mount` (`cleanup` has its own message: `cannot cleanup outside a stable or reactive scope`) | Wrap the call in `root` or `mount` |
| `cannot create a new reactive scope inside another reactive scope` | `effect` or `derive` directly inside an effect | Move it up, or wrap it in `untrack` (`show`, `indexes` and friends create effects too, so they must be called from a stable scope) |
| `attempt to yield in reactive scope` | A yield inside an effect or derive | Do the async work outside and write a source when it completes |
| `attempt to destroy an active scope` | Destroying a root from an effect inside it | Defer the destroy with `task.defer` or do it from outside |
| `root already destroyed` | Calling a destructor twice | Guard the call or destroy once |

**Async work stays outside the scope**

```luau
-- errors in strict mode: the effect yields while the remote call waits
effect(function()
	local stats = fetchStats:InvokeServer()
	label.Text = "Level " .. stats.level
end)

-- instead: do the work outside, hand the result back through a source
local level = source(1)
task.spawn(function()
	level(fetchStats:InvokeServer().level)
end)
effect(function()
	label.Text = "Level " .. level()
end)
```

When something does go wrong inside a scope, strict mode rethrows it with a trace that points at your callback rather than at Vide's internals, and a root whose function errors is destroyed before the error reaches you (its cleanups run). Read the first line of the message: the five above account for nearly every scope error.

**Key points**

- Strict mode runs reactive scopes twice, forbids yields, catches duplicate values in `values` and improves traces.
- It is on in Studio (O0/O1) and off in production (O2); the sandbox starts with it off.
- The scope errors all mean the same thing: an effect needs a stable scope around it and must not nest in another effect.
