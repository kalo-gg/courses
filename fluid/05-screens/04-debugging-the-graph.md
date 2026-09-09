# Debugging the graph

*Read Fluid's messages closely enough to find the scope you forgot or the yield you did not mean.*

Fluid · Building Real Screens · 11 min

---

Almost every Fluid bug is one of three things: something was built outside the scope it needed, a reactive callback yielded, or an instance outlived the graph that made it. The first two announce themselves — if you know what the messages look like. This lesson is that vocabulary.

| What you see | What happened | Fix |
| --- | --- | --- |
| `cannot call <name> without a parent scope` | You called `derive`, `effect`, `spring`, `interval`, `for_values` … at the top level | Wrap it in `fluid.root` |
| `cannot call <name> while the parent scope is reactive - this might be a component. consider using untrack() while creating the components.` | You built a component or a derive from inside an `effect`, a `derive` or a property function | `fluid.untrack(function() … end)` around the construction |
| `reactive scope error thing, TODO error message` | A signal-valued property or `fluid.changed` outside any scope | The same: give it a `root`. The message is simply unfinished upstream |
| `cannot cleanup outside a stable or reactive scope` | `fluid.cleanup` with no scope around it | Move it inside the root |
| `cannot cleanup given object` | `fluid.cleanup` on something with no destructor, disconnect or `Destroy` | Pass a function instead |
| `tried tweening <type>` | `fluid.spring` on a type it cannot pack — `Vector2`, `Rect`, `CFrame` | Use `lerp`, or animate a number |
| `Can't switch the data types of a spring` | A spring's input changed type after creation | One spring per type |
| `attempt to call a nil value`, from inside the animation step, every frame | A `lerp` on `UDim2` or `CFrame` | `spring`, or lerp a number |

## Warnings are where the real bugs are

Fluid errors loudly the first time and quietly afterwards. A callback that yields on its **first** run raises where you created it. The same callback yielding on a **later** run only warns: the run is abandoned half-finished, the value it should have produced is stale, and the game carries on. An `interval` whose callback errors goes quiet entirely until a call succeeds.

**A warning, and the run that never finished**

```luau
local destroy = fluid.root(function()
	local n = fluid.source(0)
	local runs = 0

	fluid.effect(function()
		runs += 1
		local value = n()
		if runs > 1 then
			coroutine.yield() -- stands in for task.wait / :InvokeServer
		end
		print("completed run", runs, "value", value)
	end)

	n(1) --> [fluid] warning; the print below never happens
	print("still running, runs =", runs) --> 2
end)

destroy()
```

The first run completes and prints. The second warns and stops mid-callback, and nothing downstream knows. In this sandbox the warning's text is empty, because tracebacks are stubbed out — in Studio it carries a stack.

> **`_G.__DEV__` does nothing**
>
> The README advertises it as the switch for development mode "at the cost of performance". In the shipped source `dev_mode = true` is hardcoded in both `src/reactive/graph.luau` and `src/logging.luau`, and `__DEV__` is never read anywhere. Every consumer pays the dev-mode cost — a coroutine per reactive evaluation — whether the flag is set or not.

One tool is worth knowing: `fluid.graph` is exported, and `fluid.graph.get_scope()` returns the scope the current thread is running in, or `nil`. Printing it answers "am I inside a root?" and "am I inside something reactive?" without guessing from an error message.

**Asking where you are**

```luau
print(fluid.graph.get_scope())              --> nil

local destroy = fluid.root(function()
	local scope = fluid.graph.get_scope()
	print(scope ~= nil, scope.effect ~= false) --> true  false  (stable)

	fluid.effect(function()
		local inner = fluid.graph.get_scope()
		print(inner ~= nil, inner.effect ~= false) --> true  true   (reactive)
	end)
end)

destroy()
```

`nil` means no scope; `scope.effect ~= false` means the scope is reactive — the two states the two different error messages are about.

**Key points**

- `cannot call <name> without a parent scope`: no root. `… while the parent scope is reactive`: a root, but a reactive one — use `untrack`.
- `reactive scope error thing, TODO error message` is the same problem with an unfinished message.
- A yield on the first run errors; on a later run it only warns, and leaves a stale value behind.
- `_G.__DEV__` is never read: dev mode is always on, and always costs you a coroutine per evaluation.
- `fluid.graph.get_scope()` tells you whether you are in a scope and whether it is reactive.
