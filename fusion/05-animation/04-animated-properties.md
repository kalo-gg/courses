# Animated properties

*Give an animation a Computed goal and hover, selection and health all animate themselves.*

Fusion · Animation · 12 min

---

Everything so far set a goal by hand. The last step is to stop doing that. Because a goal is `UsedAs`, it can be a `Computed` — and then the animation is not something you trigger, it is something that follows from your state.

A button that grows on hover is a boolean, a computed size, a spring, and no event handler that animates anything.

**Hover, derived**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent = Fusion.Children, Fusion.OnEvent

local hovered = scope:Value(false)

local size = scope:Computed(function(use)
	return if use(hovered) then 28 else 20
end)

local button = scope:New "TextButton" {
	Text = "Buy",
	TextSize = scope:Spring(size, 30, 1),
	[OnEvent "MouseEnter"] = function() hovered:set(true) end,
	[OnEvent "MouseLeave"] = function() hovered:set(false) end,
}

button.MouseEnter:Fire()
step(30)
print(math.floor(button.TextSize)) --> 27
```

The handlers set a boolean and stop. Everything after that — the computed, the spring, the property — is a consequence.

## Springs over the other types

A spring animates anything in Fusion's `Animatable` list — numbers, `UDim`, `UDim2`, `Vector2` and `Vector3`, `Color3`, `Rect`, `NumberRange` and more — because each of those can be written as a vector of numbers. So a panel's `Position`, a frame's `BackgroundColor3` and a label's `TextTransparency` all animate with the same three arguments.

**A panel that slides and a color that fades**

```luau
local open = scope:Value(false)

local goalPosition = scope:Computed(function(use)
	return if use(open) then UDim2.fromScale(0.5, 0.5) else UDim2.fromScale(0.5, 1.4)
end)

local panel = scope:New "Frame" {
	Position = scope:Spring(goalPosition, 25, 1),
	BackgroundColor3 = scope:Spring(
		scope:Computed(function(use)
			return if use(open) then Color3.fromRGB(40, 40, 60) else Color3.fromRGB(20, 20, 20)
		end),
		18, 1
	),
}

open:set(true)
step(60)
print(panel.Position, panel.BackgroundColor3)
```

Two springs, one boolean. Note that these are springs, not tweens: in this sandbox a `Tween` over `UDim2` or `Color3` errors, while a `Spring` over either runs.

> **Do not animate what you can derive**
>
> If you find yourself calling `:set` on a goal from three different handlers, the goal wants to be a `Computed` over the state those handlers were already writing. The derived version cannot get out of step with the interface, because there is only one place the target is decided.

**Key points**

- A `Computed` goal turns animation from something you trigger into something that follows from state.
- Handlers write state; the goal derives the target; the spring or tween decides the feel; the property reads.
- A spring animates every type in Fusion's `Animatable` list, including `UDim2` and `Color3`.
- A spring whose goal moves mid-flight keeps its momentum, which is what makes interrupted motion feel right.
- In this sandbox, `Tween` is for numbers and `UDim`; `Spring` covers the rest.
