# Observers

*The one place a change is allowed to do something: onChange for later changes, onBind to run now as well.*

Fusion · State Objects · 10 min

---

Computeds work out values. Observers make things happen. An `Observer` watches a state object and runs your callback when it changes — the escape hatch from Fusion's world into sounds, remotes, analytics and prints. An observer holds no value of its own; it is a subscription, and whatever the callback needs it reads for itself with `peek` at the moment it runs.

**Watching a value**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek

local health = scope:Value(100)

local observer = scope:Observer(health)

local disconnect = observer:onChange(function()
	print("health is now", peek(health))
end)

health:set(80) --> health is now 80
disconnect()   -- stop listening, without destroying the observer
health:set(60) -- (nothing printed)
```

`onChange` returns a function that disconnects that one listener. The observer itself belongs to the scope.

## Two methods

There is a second method. `onBind` does everything `onChange` does, and also runs the callback once straight away.

It is the right choice when the callback is what brings something into sync in the first place — you want it to run now *and* on every later change.

| Method | Runs now? | Runs on change? | Use when |
| --- | --- | --- | --- |
| `onChange` | No | Yes | Reacting to a change: play a sound, send a remote. |
| `onBind` | Yes | Yes | Keeping something in sync, starting from the current value. |

**onBind, and letting go of the observer**

```luau
local coins = scope:Value(50)

-- runs immediately, then on every change
scope:Observer(coins):onBind(function()
	print("coins:", peek(coins)) --> coins: 50
end)

coins:set(75) --> coins: 75

-- the observer belongs to the scope, so this is all the teardown there is
Fusion.doCleanup(scope)
coins:set(10) -- (nothing printed: the observer is gone with its scope)
```

Two ways to stop listening, and they are not the same. The function `onChange`/`onBind` returns drops one listener and leaves the observer alive; cleaning the scope up destroys the observer itself.

> **Note what a computed cannot do**
>
> `print` inside a `Computed` looks like it works, right up until Fusion decides nobody is reading that computed and skips the recalculation. Then your log goes quiet and the bug is baffling. Printing is a side effect; it belongs here.

**The division of labor**

- `Value` — state you write to, with `:set`.
- `Computed` — state worked out from other state, pure and immediate.
- `Observer` — where something is allowed to happen.
- `use` subscribes, `peek` does not. Inside a computed, use `use`.
