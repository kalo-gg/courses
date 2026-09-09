# Contextual

*Pass a theme or the local player down ten components without threading it through ten props.*

Fusion · Shipping a Fusion App · 11 min

---

A theme, the local player, a scale factor: values that most of the tree needs and nobody wants to thread through every call. `Contextual` is Fusion's answer — the source calls it "time-based contextual values, to allow for transparently passing values down the call stack".

**Setting a value for a call**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children, OnEvent = Fusion.peek, Fusion.Children, Fusion.OnEvent

local theme = Fusion.Contextual("light")

print(theme:now()) --> light

theme:is("dark"):during(function()
	print(theme:now()) --> dark
end)

print(theme:now()) --> light again
```

`:is(value):during(callback)` sets the value for the duration of the call and restores it afterwards, even if the callback errors.

"Time-based" is the important word. The value is not attached to a component or a scope — it is whatever was set at the moment `:now()` is called. Anything your callback calls, however deep, sees it; anything running later does not.

> **It does not survive a different coroutine**
>
> Because it is scoped to a call, a value set with `:during` is gone once that call returns. Reading it later — in an event handler that fires a second afterwards, say — gives you the default, not the value you set. Read it while you are building, and capture what you need.

## What belongs in one

| Value | Contextual? | Why |
| --- | --- | --- |
| A theme or palette | Yes | Every component wants it and none of them decide it |
| The local player | Yes | Fixed for the session, needed deep in the tree |
| A UI scale factor | Yes | Same reason, and it changes rarely |
| The item this row is showing | No — a prop | It differs per component, which is what props are for |
| The coin count | No — a `Value` in a state module | It changes constantly and things need to watch it |

The line between a contextual and a prop is who decides the value. If the caller is choosing it for this one component, it is a prop. If it is ambient — true of the whole screen, the whole session, the whole player — a contextual saves threading it through every function between the top and the component that needs it. Neither is a home for state that changes; that is what a `Value` is for.

**Read while building, keep what you found**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children, OnEvent = Fusion.peek, Fusion.Children, Fusion.OnEvent

local theme = Fusion.Contextual({ text = Color3.fromRGB(20, 20, 20) })

local function Button(scope, props)
	local palette = theme:now() -- read now, while this component is being built
	return scope:New "TextButton" {
		Text = props.Text,
		TextColor3 = palette.text,
		[OnEvent "Activated"] = function()
			print(palette == theme:now())
		end,
	}
end

local dark = { text = Color3.fromRGB(240, 240, 240) }

local button = theme:is(dark):during(function()
	return Button(scope, { Text = "Buy" })
end)

print(button.TextColor3 == dark.text) --> true: built while dark was set
click(button)                         --> false: :during returned long ago
```

One read at build time, kept in a local, and the component is correct forever. The handler runs after `:during` has restored the default, so the same `theme:now()` call gives a different answer — which is the whole of the lifetime rule in one file.

**Key points**

- `Fusion.Contextual(default)` carries an ambient value down the call stack without props.
- `:now()` reads the value at this moment; `:is(v):during(fn)` sets it for the length of `fn` and restores it afterwards, even on an error.
- It is time-based, not object-based: a handler running later sees the default.
- Read it while building and capture what you need.
- Ambient and unchanging is a contextual; per-component is a prop; changing is a `Value`.
