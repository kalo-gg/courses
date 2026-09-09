# Components

*A component is a function taking a scope and some props. Here is how to make yours read like Fusion's.*

Fusion · Shipping a Fusion App · 10 min

---

Fusion has no component system, because it does not need one. A component is a function that takes a scope and a table of props and returns an instance. That is the whole convention.

**A button component**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children, OnEvent = Fusion.peek, Fusion.Children, Fusion.OnEvent

local function Button(scope, props)
	return scope:New "TextButton" {
		Text = props.Text,
		Size = props.Size or UDim2.fromOffset(120, 40),
		[OnEvent "Activated"] = props.OnClick,
	}
end

local buy = Button(scope, { Text = "Buy", OnClick = function() print("bought") end })
```

Taking the scope as the first argument is not decoration: it is what lets the caller decide how long your component lives.

## Making it read like Fusion

`scoped` accepts more than one table, and merges them all into the scope's methods. Pass your own component library alongside `Fusion` and your components gain the same call syntax as the built-in constructors.

**Merged into the scope**

```luau
local components = {}

function components.Button(scope, props)
	return scope:New "TextButton" { Text = props.Text }
end

local scope = Fusion.scoped(Fusion, components)

local buy = scope:Button { Text = "Buy" } -- reads like scope:New
```

The method call passes the scope as the first argument for you, which is exactly the signature the convention already asked for.

> **Take the scope, do not make one**
>
> A component that calls `scoped(Fusion)` itself owns its instances forever, and the caller has no way to clean them up. Take the scope you are given. If a component genuinely needs a shorter life than its caller — a menu that opens and closes — that is what `innerScope` is for.

Two conventions are worth adopting on day one because they are hard to retrofit. Take the scope as the first argument and props as the second, always in that order, so `scoped` can merge your library. And give props the same freedom Fusion's own properties have: treat them as `UsedAs`, so a caller can pass a constant today and a `Computed` tomorrow without you changing anything. One name to watch while you are at it: `scoped(Fusion, components)` merges the two tables and errors on any key they share, fatally — so a component called `New`, `Value`, `Computed`, `Safe` or `version` stops the script with `Multiple definitions for 'New' found while merging.` rather than shadowing anything.

**Key points**

- A component is a function `(scope, props) -> Instance`. There is no component system to learn.
- `Fusion.scoped(Fusion, myComponents)` merges your functions in, so `scope:Badge { … }` reads like `scope:New`.
- Take the scope you are given; a component that makes its own takes the caller's cleanup away.
- Treat props as `UsedAs` so a caller can pass a constant or a state object.
- A component that needs a shorter life than its caller uses `innerScope`, not a new root scope.
