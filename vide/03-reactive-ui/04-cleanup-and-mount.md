# cleanup and mount

*Tie connections, instances and threads to scopes, and mount a whole app in one call.*

Vide · Reactive UI · 10 min

---

When a scope reruns or is destroyed, anything it created should go with it: connections it made, instances it holds, threads it spawned. `cleanup(x)` queues that. Pass a function and it is called; pass a connection and it is disconnected; pass an instance and it is destroyed; pass a thread and it is canceled with `task.cancel`.

```luau
local cleanup = vide.cleanup

local count = source(0)

local destroy = root(function()
	effect(function()
		local x = count()
		cleanup(function() print("cleaning up", x) end)
	end)

	cleanup(function() print("root destroyed") end)
end)

count(1)  -- "cleaning up 0"  (the effect reruns, so its old cleanup runs first)
count(2)  -- "cleaning up 1"
destroy() -- "root destroyed" then "cleaning up 2"
```

A cleanup queued inside an effect runs before each rerun and on destruction. One queued in a stable scope runs only on destruction. On destruction a scope runs its own cleanups first, then destroys the scopes it owns.

Register a resource the moment a component creates it: connect a signal, `cleanup(connection)`; spawn a thread, `cleanup(thread)`. Then whoever destroys the scope — a `root` destructor, `show` when its condition turns false, `indexes` when an item leaves, `unmount` — gets the teardown for free, and the component never needs a `destroy` method of its own.

> **Instances rarely need cleanup**
>
> A Roblox instance is freed once nothing references it and it has no parent, so a component's instances usually die with the UI they are in. The exception is a handler that references its own instance (`button.Activated:Connect(function() button.Text = … end)`): that cycle keeps the instance alive, so disconnect the connection with `cleanup(connection)` or destroy the instance explicitly.

## mount

`mount(component, target)` is `root` for whole screens: it runs the component in a new stable scope, parents what it returns to `target`, and returns the destructor. This is the usual entry point of a Vide app, called once from a LocalScript with `PlayerGui` as the target. Calling the destructor stops every effect in the app; the instances stay where they are unless the app queued them with `cleanup(gui)` or you destroy them yourself.

```luau
local mount = vide.mount

local function App()
	return create "ScreenGui" {
		Counter {},
	}
end

local unmount = mount(App, Players.LocalPlayer.PlayerGui)

-- later, e.g. when the player leaves the menu
unmount()
```

**Key points**

- `cleanup` ties functions, connections, instances and threads to the current scope's lifetime.
- Inside an effect, cleanups run before every rerun as well as on destruction.
- `mount(App, target)` runs the app in a stable scope, parents it, and returns the destructor; destroying stops effects but leaves instances in place.
