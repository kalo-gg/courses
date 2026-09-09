# Events and properties

*OnEvent listens, OnChange watches a property, and Out streams a property back into state.*

Fusion · Instances · 11 min

---

Three special keys connect an instance to the world. All of them belong to the scope, so the connections are dropped when the scope is cleaned up — you never disconnect them by hand.

**Listening**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent, OnChange, Out = Fusion.Children, Fusion.OnEvent, Fusion.OnChange, Fusion.Out
local Attribute, AttributeOut, AttributeChange = Fusion.Attribute, Fusion.AttributeOut, Fusion.AttributeChange

local clicks = scope:Value(0)

local button = scope:New "TextButton" {
	Text = "Click me",
	[OnEvent "Activated"] = function()
		clicks:set(peek(clicks) + 1)
	end,
}
```

`OnEvent` names any event on the instance. The handler is ordinary Luau, so `peek` is the right reader here.

## Reading the instance back

Sometimes the instance is the source of truth: a `TextBox` the player types into, a property Roblox changes on its own. Fusion has two keys for that direction, and the difference between them is whether you want a callback or a state object.

`OnChange` runs a callback when a property changes, and `Out` goes further — the source calls it a key that "allows users to extract values from an instance into an automatically-updated `Value` object". Point it at a `Value` and the property writes itself into your state.

**OnChange versus Out**

```luau
local typed = scope:Value("")

scope:New "TextBox" {
	-- a callback when it changes
	[OnChange "Text"] = function(newText)
		print("they typed", newText)
	end,

	-- or stream it into state, no callback needed
	[Out "Text"] = typed,
}
```

`Out` is one way: the instance drives the `Value`, and anything computed from that value follows. Setting the value does not move the property back; bind the property as well (`Text = typed`) when you need both directions.

| Key | Direction | Gives you |
| --- | --- | --- |
| `Text = value` | state → instance | The property follows the state. |
| `[OnChange "Text"]` | instance → callback | A function call, with the new value. |
| `[Out "Text"]` | instance → state | A `Value` that updates itself. |
| `[OnEvent "Activated"]` | instance → callback | A connection owned by the scope. |

**Key points**

- `[OnEvent "Name"]` connects any event on the instance; the connection belongs to the scope.
- `[OnChange "Prop"]` runs a callback with the new value when the property changes.
- `[Out "Prop"] = value` writes the property into a `Value` as it changes; it must be a `Value`, and anything else is a **fatal** `invalidOutType`.
- `Out` is one-way. For two-way, bind the property too.
