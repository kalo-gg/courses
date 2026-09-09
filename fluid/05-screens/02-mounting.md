# Mounting a screen

*Bind a component onto a ScreenGui that already exists, and know what unmounting does not do.*

Fluid · Building Real Screens · 10 min

---

`fluid.mount` runs a component inside a fresh `root` and binds what it builds onto an instance that already exists — a `ScreenGui` from Studio, usually. It is three lines of source: `root(function() bind(inst, { fn() }) end)`. It returns that root's destructor, so the mount and its teardown come as a pair.

**mount**

```luau
-- in a game: local screenGui = playerGui:FindFirstChild("HUD")
local screenGui = Instance.new("ScreenGui")

local unmount = fluid.mount(function()
	return fluid.create("Frame")({
		fluid.create("TextLabel")({ Text = "Hello" }),
	})
end, screenGui)

print(#screenGui:GetChildren()) --> 1

unmount()
print(#screenGui:GetChildren()) --> still 1: the graph stopped, the Frame did not
```

> **Unmounting stops the graph, not the instances**
>
> `mount`'s destructor is an ordinary root destructor, with the ordinary limitation: bindings stop, effects stop, event connections are disconnected — and every instance it parented into your `ScreenGui` is still sitting there. If a mount is something you tear down and rebuild, destroy the mount target's children yourself, or hand the top of the tree to `fluid.cleanup` inside the component.

## Where a screen goes

The component you hand `mount` is called inside that new root, so it is free to create derives, effects and reactive properties — but it is a fresh scope, not the caller's, so anything it registers is torn down by the returned destructor rather than by an outer one.

**Into PlayerGui, and back out again**

```luau
local Players = game:GetService("Players")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RoundEnd"
screenGui.Parent = Players.LocalPlayer.PlayerGui

local unmount = fluid.mount(function()
	local panel = fluid.create("Frame")({ Name = "Panel" })
	fluid.cleanup(panel) -- the mount's own scope owns it now
	return panel
end, screenGui)

print(screenGui:FindFirstChild("Panel") ~= nil) --> true
unmount()
print(screenGui:FindFirstChild("Panel") ~= nil) --> false
```

One `cleanup` inside the component is the difference between `unmount()` meaning "stop" and meaning "remove".

Two ways to get a screen on the screen, and they differ only in ergonomics. `mount(fn, inst)` opens a root, calls the component inside it, and binds the result onto an instance you already have. `root(fn)` plus `panel.Parent = …` does the same by hand and hands you the instance as a second return value, which is easier to inspect.

**Key points**

- `fluid.mount(fn, inst)` opens a root, runs `fn` in it, and binds the result onto an existing instance.
- It returns that root's destructor, so mounting and unmounting come as a pair.
- Unmounting stops the graph and leaves the instances parented, exactly like any root.
- `fluid.cleanup(top)` inside the component makes `unmount()` remove the tree.
- `root` + `Parent` does the same job and hands you the instance back, which is easier to test against.
