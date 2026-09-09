# New and Children

*Build a whole tree of instances in one expression, and bind any property to a constant or a state object.*

Fusion · Instances · 10 min

---

Everything so far built one instance at a time. A real screen is a tree: a frame holding a title, a button and a row of icons, each with its own properties.

`New` builds a whole subtree in one expression, and `[Children]` is the key that assembles it. `New` is a method on the scope, so every instance in that tree is owned by the scope before it is ever parented.

**An instance with children**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent, OnChange, Out = Fusion.Children, Fusion.OnEvent, Fusion.OnChange, Fusion.Out
local Attribute, AttributeOut, AttributeChange = Fusion.Attribute, Fusion.AttributeOut, Fusion.AttributeChange

local menu = scope:New "Frame" {
	Name = "Menu",
	Size = UDim2.fromOffset(300, 200),

	[Children] = {
		scope:New "TextLabel" { Text = "Shop" },
		scope:New "TextButton" { Text = "Buy" },
	},
}

print(#menu:GetChildren()) --> 2
```

`[Children]` takes one instance, an array of them, or a state object producing either.

The source describes `[Children]` as the key that "parents any given descendants into an instance". *Descendants*, plural and nested: an array inside an array is flattened, and a state object holding instances is unwrapped and re-parented when it changes. That last part is how a list that grows and shrinks works, which Tables & Lists builds on.

> **Properties take `UsedAs`**
>
> Every property accepts `UsedAs<T>` — "a `T`, or something that produces a `T`". `Text = "Shop"` and `Text = someComputed` are both valid, and that single rule is why state and constants are interchangeable everywhere in a property table.

## Properties that are state

**A constant and a state object, side by side**

```luau
local title = scope:Value("Shop")

local header = scope:New "TextLabel" {
	Name = "Header",
	Text = title,     -- follows the value
	TextSize = 24,    -- a constant
	[Children] = scope:New "UIPadding" {},
}

title:set("Sale!")
print(header.Text) --> Sale!
```

A property bound to a `Value` follows it; a `Computed` works the same way. Nothing in the tree has to be told.

**Key points**

- `scope:New "Class" { … }` is curried: scope and class first, property table second.
- `[Children]` parents an instance, an array (flattened) or a state object holding either.
- Every property takes `UsedAs<T>`: a constant, or a state object producing one.
- Build the tree in one expression; let the data change it afterwards.
