# Computeds

*Work a value out from other state with use, and keep the calculation pure and immediate.*

Fusion · State Objects · 11 min

---

A `Computed` takes a callback and hands it a reader called `use`. Calling `use(something)` returns that thing's value **and** records that this computed depends on it. When a dependency changes, the computed throws its result away and works it out again.

**Derived state**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek

local numCoins = scope:Value(50)
local itemPrice = scope:Value(10)

local finalCoins = scope:Computed(function(use)
	return use(numCoins) - use(itemPrice)
end)

print(peek(finalCoins)) --> 40

itemPrice:set(20)
print(peek(finalCoins)) --> 30
```

Two dependencies were recorded, so changing either one is enough to make the result stale.

## Two rules

Fusion is allowed to postpone or skip a recomputation while nothing is reading the result. That single fact is where both rules about computeds come from.

> **A computed's callback must**
>
> - **Be immediate.** It must never yield — no `task.wait`, no waiting on a remote. Fusion needs the answer now.
> - **Have no side effects.** The documentation's own example is playing a sound: if the recalculation is skipped, the sound never plays, and if it runs twice you hear it twice.

> **Side effects go in an Observer**
>
> If you want something to *happen* when state changes — a sound, a remote call, a print — that is the next lesson's job. A computed only works out a value.

## Chains and the second argument

**Computeds read computeds**

```luau
local price = scope:Value(100)
local quantity = scope:Value(2)

local subtotal = scope:Computed(function(use)
	return use(price) * use(quantity)
end)

local label = scope:Computed(function(use)
	return "Total: " .. use(subtotal) -- a computed is state too
end)

quantity:set(3)
print(peek(label)) --> Total: 300
```

A computed depends on whatever it `use`d, computeds included. Change `quantity` and both recalculate, in order.

The callback is actually handed two things: `use`, and a **scope** of its own. Anything you create with that scope lives until the computed recalculates, when Fusion cleans it up and gives the next run a fresh one. Most computeds never touch it; the ones that build instances do, and that is the last lesson of this module.

**Key points**

- `scope:Computed(function(use) … end)` derives a value; `use` reads and records.
- Computeds depend on computeds; a chain recalculates in dependency order.
- The callback must be immediate and must not have side effects, because Fusion may postpone or skip it.
- The callback's second argument is a scope that is cleaned up on every recalculation.
