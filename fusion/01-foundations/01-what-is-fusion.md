# What is Fusion?

*Describe the interface for a given state and let Fusion keep every instance in sync.*

Fusion · Fusion Foundations · 8 min

---

A shop menu shows a coin balance. You buy something, so a script sets `CoinLabel.Text`. Later the same balance appears on the HUD, so that script sets a second label too. Then a sale halves the prices, and every place that displayed a price needs finding again.

That bookkeeping is the problem **Fusion** removes. You write down what the interface should look like for a given state, and Fusion updates the instances when the state changes. Nobody has to remember which labels exist.

## By hand, then with Fusion

**By hand: every writer of the state also updates the UI**

```luau
local coins = 0

local label = Instance.new("TextLabel")
label.Text = "Coins: 0"
label.Parent = hud

local function addCoins(n)
	coins += n
	label.Text = "Coins: " .. coins -- and again, everywhere else coins is shown
end
```

**With Fusion: the label describes itself**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)

local coins = scope:Value(0)

scope:New "TextLabel" {
	Parent = hud,
	Text = scope:Computed(function(use)
		return "Coins: " .. use(coins)
	end),
}

local function addCoins(n)
	coins:set(Fusion.peek(coins) + n) -- the label follows on its own
end
```

`use(coins)` both reads the value and records that this text depends on it. Change the value and only that text recomputes.

## Three ideas, one library

Fusion calls itself "a portable Luau companion library for simpler, more descriptive code". Reactivity is the headline, but the design goes wider than UI: the state objects and the scope system are plain Luau and do not need Roblox instances to work.

The course follows that split: memory first, then state, then the part that touches Roblox. Each row below is a lesson, in the order you will meet them.

| Idea | In Fusion 0.3 | You will meet it in |
| --- | --- | --- |
| Something that owns objects | `Fusion.scoped(Fusion)` | Scopes own what you create |
| A value you can write to | `scope:Value(0)` | Values |
| A value derived from other values | `scope:Computed(function(use) … end)` | Computeds |
| A side effect when state changes | `scope:Observer(thing)` | Observers |
| An instance | `scope:New "TextLabel" { … }` | Your first interface |
| Motion | `scope:Spring(goal)`, `scope:Tween(goal)` | Tweens and Springs |

> **Fusion does not replace Roblox UI**
>
> You still get ordinary `Frame`s and `TextLabel`s, still lay them out with `UIListLayout` and `UIPadding`, and still design in Studio if you want — `Hydrate` binds Fusion state onto instances that already exist.

**Key points**

- Fusion updates instances from state; you never write `label.Text = …` after a change again.
- `use` reads a value and records the dependency; that record is what makes the update happen.
- Every object is created inside a scope, which is the first argument of every constructor.
- Memory, state and instances are the three parts of the library, in that order.
