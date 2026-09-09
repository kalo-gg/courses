# Laying out an app

*A state module, a folder of components, one owning scope: the shape the final project builds.*

Fusion · Shipping a Fusion App · 11 min

---

Everything in this course has been one file. A shipped screen is not, and the split is always the same three parts.

A **state module** holding the `Value`s and `Computed`s and the functions that write them; a folder of **components**, each a function taking a scope; and one **entry script** that owns the root scope, mounts the tree and is the only file that knows about `PlayerGui`.

**The layout**

```text
StarterPlayerScripts/
  main.luau            -- the LocalScript: owns the scope, mounts, cleans up
  state.luau           -- Values, Computeds, and the functions that write them
  theme.luau           -- a Contextual, if the screen has one
  components/
    ShopPanel.luau     -- (scope, props) -> Instance
    ItemRow.luau
    DetailsPanel.luau
  Packages/
    Fusion.luau        -- however Fusion was installed
```

Three kinds of file and one rule about direction: components require state, state requires nothing, and only `main` requires both.

## What a state module exports

A state module exports state objects, not values: `coins`, not `peek(coins)`. Anything that reads a snapshot has frozen it. It also exports the functions that write — `buy`, `select` — so there is exactly one place each change can happen, which is what makes a bug findable. Components never call `:set` on a state object directly; they call the function the state module named.

**state.luau, in outline**

```luau
local Fusion = require("./Packages/Fusion")
local peek = Fusion.peek
local scope = Fusion.scoped(Fusion)

local coins = scope:Value(50)
local selectedId = scope:Value(nil)

local canAfford = scope:Computed(function(use)
	return use(coins) >= 15
end)

local function buy()
	if not peek(canAfford) then return end
	coins:set(peek(coins) - 15)
end

return { scope = scope, coins = coins, selectedId = selectedId, canAfford = canAfford, buy = buy }
```

State objects out, writer functions out, no instances anywhere. Note the module's own long-lived scope: state that lasts as long as the place does should not live in the screen's scope, which comes and goes.

> **Two scopes, not one**
>
> A screen that opens and closes many times wants two lifetimes: the state module's scope, made once when the place loads, and the screen's scope, made fresh each time it opens (or an `innerScope` of a session scope). Put a `Value` the player's coins live in into the screen's scope and it is destroyed the first time they close the shop.

> **Where to go next**
>
> Fusion 0.4 is in development on `main` and changes some of this again. When it lands, the first thing to check is `Fusion.version` — and the second is whether the tutorial you are reading was written for the release you have.

**The whole course, in five lines**

- `scoped(Fusion)` makes an owner; `doCleanup` destroys everything it owns, in reverse.
- `Value` is what you write; `Computed` is what you derive; `Observer` is where things happen.
- `New` and `Hydrate` turn state into instances, and every property takes `UsedAs`.
- `ForValues` turns a table of data into a table of instances that tracks it.
- `Tween` and `Spring` are state objects with a goal, so they compose with all of the above.
