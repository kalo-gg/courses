# show and switch

*Pick one branch from state with show, or one of many with switch — both are derives underneath.*

Fluid · Lists & Control Flow · 10 min

---

`show` takes a condition and two things, and gives you back a readable holding whichever one applies. The second branch is optional — with one branch, it reads nil when the condition is false.

**One branch or two**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local loggedIn = fluid.source(false)

	local greeting = fluid.show(function()
		return loggedIn()
	end, "Welcome back", "Please log in")

	print(greeting()) --> Please log in
	loggedIn(true)
	print(greeting()) --> Welcome back
end)

destroy()
```

The condition is a function, so it can read sources and be tracked like anything else.

## Many branches: switch

`switch` is the many-branch version, and it is curried: the condition first, then a table of cases keyed by the value.

**switch**

```luau
local destroy = fluid.root(function()
	local screen = fluid.source("menu")

	local body = fluid.switch(function()
		return screen()
	end)({
		menu = "The main menu",
		shop = "The shop",
		settings = "Settings",
	})

	print(body()) --> The main menu
	screen("shop")
	print(body()) --> The shop
end)

destroy()
```

A value with no matching case reads nil, so an unhandled state shows nothing rather than erroring. `switch` builds a `derive` internally, so it needs the root.

> **Both are derives**
>
> **Both need a stable scope**, and both read their branches through `fluid.read` inside an `untrack` — so a branch may be a plain value or a function returning one, and reading it never subscribes you to anything it touched.

`switch` looks up the current value in the options table. A value with **no entry** reads `nil` rather than erroring, which is usually what you want for an unhandled state and occasionally hides a typo: a `switch` on `"Shop"` against a table keyed `shop` shows nothing, silently. If the state has a default, give the table a case for it.

**Key points**

- `fluid.show(cond, truthy, falsy?)` swaps between two things; with one branch the false case is `nil`.
- `fluid.switch(cond)(cases)` is curried and keyed by the condition's value.
- An unmatched key reads `nil`; nothing errors.
- Both build a `derive`, so both need a stable scope.
- Both accept a value **or** a function per branch, and read it untracked.
