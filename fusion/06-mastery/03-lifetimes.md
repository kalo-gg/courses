# Reading Fusion's warnings

*possiblyOutlives, poisoned scopes and destroyedTwice: what Fusion tells you before it becomes a bug.*

Fusion · Shipping a Fusion App · 12 min

---

Fusion talks to you. Almost every lifetime mistake produces a message in the output before it produces a bug, and the messages are specific enough to name the fix.

Three of them account for nearly everything, and knowing them turns a class of confusing UI failures into a class of one-line corrections.

**The warning you will meet most**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children, OnEvent = Fusion.peek, Fusion.Children, Fusion.OnEvent

local session = Fusion.scoped(Fusion)
local menu = Fusion.innerScope(session)

-- the Value dies with the menu; the Computed lives as long as the session
local coins = menu:Value(10)
local caption = session:Computed(function(use)
	return "Coins: " .. use(coins)
end)

print(peek(caption))
--> [Fusion] The use()-d Value will be destroyed before the Computed; the latter
--> is in a different scope that gets destroyed too quickly. To fix this, review
--> the order they're created in, and what scopes they belong to. …
```

`possiblyOutlives` is not saying the values are wrong today. It is saying that when the menu closes, the computed will still hold a dependency on something destroyed. The fix is always to move one of them, never to silence it.

## The ones you will actually see

| Message | What happened | Fix |
| --- | --- | --- |
| `possiblyOutlives` (warning) | Something depends on an object that will be destroyed first | Move one of them into the other's scope |
| `Attempted to use a scope after it's been destroyed` (fatal) | A scope was reused after `doCleanup` poisoned it | A fresh scope per open, or an `innerScope` |
| `useAfterDestroy` (fatal) | An object was read after its scope was cleaned up | The reader outlived what it reads: the same fix as `possiblyOutlives`, one step later |
| `destroyedTwice` (fatal) | `doCleanup` was given something it is already cleaning up: one task owned by two scopes | Own each object once; an `innerScope` unregisters itself when destroyed early, so it is safe either way |
| `Error in callback` (`callbackError`) — sometimes fatal | A callback threw. A `Computed` or a `For` processor is wrapped in `xpcall`, so Fusion logs this non-fatally and keeps the previous value. Two callers are not: an `Observer` callback is invoked bare through `doTaskImmediate`, so a throw inside one propagates and kills the script, and `Contextual:during` catches its callback's error only to re-raise it with `logError`, which is fatal | Fix the callback. Only the Computed and For cases carry on without you |

One rule prevents most of them: an object should be created in the scope that matches how long it is needed, and read only by things that die no later. When those two disagree, one of the messages above fires. When a designer-built instance is involved, remember that `Hydrate` takes ownership — hydrate a `:Clone()` when the original must survive the scope.

**A poisoned scope, in the output**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children, OnEvent = Fusion.peek, Fusion.Children, Fusion.OnEvent

local screen = Fusion.scoped(Fusion)
screen:New "Frame" { Name = "Panel" }

Fusion.doCleanup(screen)

local ok, err = pcall(function()
	return screen:Value(0) -- the menu opens a second time
end)

print(ok)  --> false
print(err) --> [Fusion] Attempted to use a scope after it's been destroyed;
           --  `doCleanup()` was previously called on this scope. Ensure you are
           --  not reusing scopes after cleanup. …
```

`doCleanup` empties the array and then replaces the table's metatable, so the scope does not merely look empty afterwards — every later read or write on it throws. This one is fatal, not a warning, which is why the open-and-close bug shows up as a crash on the second open rather than as a leak.

> **Nothing checks the absence of a warning**
>
> These are printed, not thrown, so a screen with a lifetime bug runs and looks fine in testing. Read the output window while you develop: there is no assertion that catches this for you, and no way to test for a warning that did not happen.

**Key points**

- `possiblyOutlives` warns that something will be destroyed before what depends on it.
- A cleaned-up scope is poisoned; touching it is the fatal `Attempted to use a scope after it's been destroyed`.
- `useAfterDestroy` is `possiblyOutlives` one step later, and it is fatal.
- `destroyedTwice` means one object is owned twice; own each thing once.
- Warnings are printed, never thrown — read the output window, because nothing can prove their absence.
