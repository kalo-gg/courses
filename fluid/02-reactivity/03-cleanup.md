# Cleanup and ownership

*A root tears down the graph it owns; your instances only go if you hand them over.*

Fluid · The Reactive Graph · 10 min

---

`root` returns a destructor, and calling it tears down the **reactive graph** created inside: effects stop and derives are dropped. What it does **not** do is destroy your instances — `create` builds them and binds them, but never registers them for teardown. If you want the UI to go too, hand it to `fluid.cleanup` yourself.

**Registering teardown**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local someSignal = game:GetService("Players").PlayerAdded
local someInstance = Instance.new("Frame")

local destroy = fluid.root(function()
	local connection = someSignal:Connect(function() end)

	fluid.cleanup(function()
		connection:Disconnect()
	end)

	fluid.cleanup(someInstance) -- instances can be handed over directly

	print("connected:", connection.Connected) --> true
end)

destroy() -- the connection is disconnected and the instance destroyed
```

## What a destructor actually reaches

This is the mechanism behind everything else. An effect registers its own cleanup so it stops when its scope ends. `create` does not: it is `bind(Instance.new(class), props)` and nothing more, so **the instance outlives the scope unless you say otherwise**. Reaching for `cleanup` yourself is how an instance — or anything else you bring from outside Fluid — joins the arrangement.

> **A `root` alone does not destroy your UI**
>
> This is the one to remember, because nothing warns you. `destroy()` stops every effect inside the root and leaves the Frame sitting in `PlayerGui` with all its children and its original `Parent`. Screens opened and closed in a loop pile up, and the symptom is a slow leak rather than a crash. Hand the top of each tree to `fluid.cleanup` and the whole subtree goes with it.

**Handing the tree over**

```luau
local holder = Instance.new("Folder") -- pretend this is PlayerGui

local destroy = fluid.root(function()
	local panel = fluid.create("Frame")({
		Name = "Panel",
		fluid.create("TextLabel")({ Text = "Score" }),
	})

	panel.Parent = holder
	fluid.cleanup(panel) -- one line; the whole subtree goes with it
end)

print(#holder:GetChildren()) --> 1
destroy()
print(#holder:GetChildren()) --> 0
```

`cleanup` takes an instance directly and calls `:Destroy()` on it, which takes the children too.

`cleanup` accepts a function, a thread, an `RBXScriptConnection`, an instance, or any table with a `destroy`, `Destroy`, `disconnect` or `Disconnect` method. Anything else raises `cannot cleanup given object` at the point you register it.

**Key points**

- A root destructor stops the reactive graph it owns and nothing else.
- `fluid.cleanup(x)` attaches teardown to the current scope: functions, threads, connections and instances all work.
- Instances built by `create` are torn down **only** this way.
- `Activated = fn` is disconnected for you; a `:Connect` inside an `action` is not.
- A screen you open and close in a loop leaks silently unless the top of its tree is registered.
