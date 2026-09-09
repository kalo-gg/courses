# References and attributes

*Capture instances inline because :set returns its argument, and bind attributes with the same three keys as properties.*

Fusion · Instances · 11 min

---

Two things a real panel needs that the property table does not obviously give you. First, a handle on an instance you built inside a tree — the close button three levels down — without walking `GetChildren()` afterwards.

Second, data that lives on the instance itself: attributes, which Roblox saves, replicates and shows in the Properties panel, so scripts outside Fusion can read them.

**Capturing an instance with :set**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent, OnChange, Out = Fusion.Children, Fusion.OnEvent, Fusion.OnChange, Fusion.Out
local Attribute, AttributeOut, AttributeChange = Fusion.Attribute, Fusion.AttributeOut, Fusion.AttributeChange

local closeButton = scope:Value(nil)

local panel = scope:New "Frame" {
	[Children] = {
		scope:New "TextLabel" { Text = "Settings" },
		closeButton:set(scope:New "TextButton" { Text = "Close" }),
	},
}

print(peek(closeButton).Text)            --> Close
print(peek(closeButton).Parent == panel) --> true
```

`:set` returns the value you passed, so one call both hands the button to `[Children]` and stores it in `closeButton`. This is what replaced 0.2's `[Ref]` key.

## Attributes

Attributes get the same three keys properties have, one word apart. `[Attribute "Name"]` writes a constant or state object into the attribute; `[AttributeOut "Name"]` streams the attribute into a `Value`; `[AttributeChange "Name"]` runs a callback with the new value. `AttributeOut` wants a `Value`, exactly as `Out` does: hand it anything else and Fusion raises `invalidAttributeOutType` — and that one is **fatal**. `AttributeOut.luau` calls `External.logError`, which calls `error()`, so the script stops on the line that built the instance rather than carrying on with a broken binding.

| Property | Attribute | Direction |
| --- | --- | --- |
| `Text = state` | `[Attribute "Health"] = state` | state → instance |
| `[Out "Text"] = value` | `[AttributeOut "Health"] = value` | instance → state |
| `[OnChange "Text"] = fn` | `[AttributeChange "Health"] = fn` | instance → callback |

**An attribute that follows state**

```luau
local health = scope:Value(10)
local seen = scope:Value(nil)

local part = scope:New "Frame" {
	[Attribute "Health"] = health,
	[AttributeOut "Health"] = seen,
	[AttributeChange "Health"] = function(new)
		print("health attribute is now", new)
	end,
}

health:set(20) --> health attribute is now 20
print(part:GetAttribute("Health"), peek(seen)) --> 20 20
```

Three keys on one attribute is unusual, but it shows the three directions at once.

**Key points**

- `value:set(x)` returns `x`, so a state object can capture an instance inline; `Ref` is gone.
- `[Attribute]`, `[AttributeOut]` and `[AttributeChange]` mirror `prop =`, `[Out]` and `[OnChange]` for attributes.
- `Out` and `AttributeOut` must be given a `Value`; giving them anything else is a fatal error, not a warning.
- Attributes are the right home for per-instance data other scripts or Studio need to see.
