# Signals and connections

*Connect a function to an event, keep the connection it returns, and know when the handler actually runs.*

Roblox Fundamentals · Events · 10 min

---

A bell in the lobby should ring whenever someone clicks it. There is nothing to check in a loop: a click is a moment, not a property. Roblox reports moments through **events**, and each one is an `RBXScriptSignal` on some instance: a part's `Touched`, a prompt's `Triggered`, a ClickDetector's `MouseClick`.

You hand an event a function, and the engine calls it each time the event fires, with the event's arguments. `MouseClick` passes the player who clicked.

## Connect, and keep the connection

**ServerScriptService/Main**

```luau
local bell = workspace.Bell

local function onClick(player: Player)
	print(player.Name .. " rang the bell")
end

local connection = bell.ClickDetector.MouseClick:Connect(onClick)
print(connection.Connected) --> true

-- later, when the bell should go quiet
connection:Disconnect()
print(connection.Connected) --> false
```

`Connect` returns an `RBXScriptConnection`. Keep it: calling `Disconnect` on it is how you stop listening.

`Connect` is one of three ways to listen. `Once` takes only the next firing: the connection is disconnected before your function runs. `Wait` yields the running thread until the event fires, then returns that firing's arguments.

```luau
local detector = workspace.Bell.ClickDetector

detector.MouseClick:Once(function(player)
	print(player.Name .. " rang first")
end)

local player = detector.MouseClick:Wait() -- this thread sleeps until a click
print("woken by " .. player.Name)
```

`Once` returns a connection too, so you can still cancel it before it fires.

## When the handler runs

Roblox recommends **deferred** signals: firing an event queues its handlers, and they run at the next resumption point, such as a `task.wait` resuming or the next frame. Template places set `Workspace.SignalBehavior` to `Deferred`, and so does this sandbox.

An older place may still say `Default`, which today means `Immediate`: the handler runs inside the code that fired the event.

```luau
local part = Instance.new("Part")
local changes = 0
part.Changed:Connect(function(property)
	changes += 1
end)

part.Transparency = 0.5
print(changes) --> 0: the handler is queued, not run
task.wait()
print(changes) --> 1
```

`Changed` fires when a property changes (next lesson). Code that fires an event and reads the handler's result on the next line gets the old value.

> **Recognising older code**
>
> Old scripts and free models write `part.Touched:connect(onTouched)` and `event:wait()`. The lowercase spellings are old aliases, and the reference documents only `Connect`, `Once` and `Wait`, so write those.

**Key points**

- An event is an `RBXScriptSignal`; its handler receives the event's arguments, such as the player for `MouseClick`.
- `Connect` returns an `RBXScriptConnection`; `Disconnect` stops it and drops any queued calls.
- `Once` listens for one firing; `Wait` yields the thread until the next one.
- With Deferred signals, which template places use, a handler runs at the next resumption point, not on the line that fired it.
