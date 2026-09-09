# Branches that linger

*Hold the outgoing branch long enough to animate it away, and read the array that comes back.*

Fluid · Lists & Control Flow · 10 min

---

`show_delay` and `switch_delay` do the same job but hold the outgoing branch for a moment before dropping it — the pause that lets an old screen animate away instead of vanishing. The name suggests you can swap them in. You cannot: the shape changes in two places at once.

|  | `show` / `switch` | `show_delay` / `switch_delay` |
| --- | --- | --- |
| A branch is | a value, or a function returning one | **always a function**, taking `active` and returning `(value, delay)` |
| The result reads | the current branch | **an array** of every branch still alive |
| During a transition | one value | two entries: the incoming one and the outgoing one |

`show_delay` is `switch_delay` underneath: it coerces the condition with `not not` and looks it up in a two-entry table keyed `true` and `false`. Anything true about one is true about the other.

**show_delay**

```luau
local destroy = fluid.root(function()
	local loggedIn = fluid.source(false)

	local panels = fluid.show_delay(function()
		return loggedIn()
	end, function(active)
		return "Welcome back", 0.3 -- value, then how long to linger
	end, function(active)
		return "Please log in", 0.3
	end)

	print(#panels(), panels()[1]) --> 1  Please log in

	loggedIn(true)
	print(#panels())              --> 2  both branches, for 0.3s
end)

destroy()
```

`active` reads false in whichever branch is on its way out, which is what the branch animates on. The array is built from a hash table, so do not rely on which index is which — iterate it.

> **Plain values do not error here**
>
> `fluid.show_delay(cond, "yes", "no")` — the call you would write if you assumed it matched `show` — is accepted and returns a function that produces nonsense. Nothing warns you. If you reach for a `_delay` variant, convert every branch to a function first.

## switch_delay, and what `active` is for

Every branch is a function taking `active` and returning `(value, delay)`. `active` is the branch's own readable: `true` while it is the chosen one, `false` from the moment the condition moves on. That is what a fade binds to, and it is why the branch has to be a function — the value has to be built with `active` in hand.

**switch_delay**

```luau
local destroy = fluid.root(function()
	local screen = fluid.source("menu")

	local panels = fluid.switch_delay(function()
		return screen()
	end)({
		menu = function(active) return "MENU", 0.5 end,
		shop = function(active) return "SHOP", 0.5 end,
	})

	print(#panels())  --> 1
	screen("shop")
	print(#panels())  --> 2   both, for half a second

	step(45)
	print(#panels())  --> 1

	screen("about")   -- no case for it
	print(#panels())  --> 1   the outgoing branch is still leaving
	step(45)
	print(#panels())  --> 0
end)

destroy()
```

An unmatched key is not an error here either — it just means no branch is incoming, so the array empties once the outgoing one has had its delay.

**Key points**

- `show_delay` / `switch_delay` keep the outgoing branch alive for a delay so it can animate away.
- Every branch must be a **function** `(active) -> (value, delay?)`; a plain value is accepted and produces nonsense.
- The result is an **array** of the live branches — one normally, two mid-transition, in no defined order.
- `active` reads `false` in the branch that is leaving, for the whole delay.
- `show_delay` is `switch_delay` with `true` and `false` for keys.
