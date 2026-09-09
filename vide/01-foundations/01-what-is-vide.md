# What is Vide?

*Declarative, reactive UI for Luau: describe the interface, let the library keep it in sync.*

Vide · Vide Foundations · 9 min

---

Picture a health bar. The player takes damage, so a script sets `Health.Text` and shrinks `Bar.Size`. Later a shield is added, so the same script also recolors the bar. Then a second screen shows health too, so the logic is copied. Every new piece of state means finding every place that displays it and updating it by hand.

**Vide** removes that bookkeeping. You write a function that describes the health bar in terms of the current state, and Vide reruns exactly the parts that depend on the values that changed.

## Imperative versus declarative

**By hand: update every property yourself**

```luau
local label = Instance.new("TextLabel")
label.Text = "count: 0"
label.Parent = screen

local count = 0
button.Activated:Connect(function()
	count += 1
	label.Text = "count: " .. count -- remember to do this everywhere count is shown
end)
```

**With Vide: describe the label once**

```luau
local count = source(0)

root(function()
	create "TextLabel" {
		Text = function()
			return "count: " .. count()
		end,
		Parent = screen,
	}
end)

button.Activated:Connect(function()
	count(count() + 1) -- the label updates itself
end)
```

The `Text` function reads `count`, so Vide reruns it whenever `count` changes. Nothing else needs to know the label exists. The `root` is not optional: a function on a property key builds a reactive node, and Vide refuses to create one with no scope to own it — Sources, Effects & Scopes is where that rule gets its own lesson.

## What Vide is made of

Vide is inspired by [Solid](https://www.solidjs.com/), a JavaScript library built on the same idea: **fine-grained reactivity**. Instead of re-rendering a whole component when something changes, only the specific property bindings that read the changed value run again.

| Idea | In Vide | You will meet it in |
| --- | --- | --- |
| An element | `create "TextButton" { … }` | this module |
| A piece of state | `source(0)` | Sources, Effects & Scopes |
| Something that reacts to state | `effect(function() … end)` | Sources, Effects & Scopes |
| A value computed from state | `derive(function() … end)` | Sources, Effects & Scopes |
| Showing or listing UI based on state | `show`, `switch`, `indexes`, `values` | Dynamic Scopes |
| Motion | `spring(source)` | Actions, Springs & Context |

## What Vide is not

Vide does not replace Roblox UI instances: it creates and updates ordinary `Frame`s, `TextLabel`s and `TextButton`s. It has no built-in components, no styling language and no virtual DOM. It is a small library (about two thousand lines) with one job: keeping instances in sync with state. Layout still comes from `UIListLayout`, `UIPadding` and friends, exactly as before.

> **Vide and Fusion**
>
> Fusion is the other popular reactive UI library for Roblox. Both share the state-and-computed model. Vide's distinguishing choices are that state is read and written by *calling* the source, effects are tracked automatically rather than declared, and lifetimes are managed by scopes instead of explicit destruction lists. Kalo has a separate Fusion course.

**Key points**

- Vide is a **reactive, declarative** UI library for Luau, inspired by Solid.
- You describe UI as a function of state; Vide reruns only the bindings that read changed values.
- It creates ordinary Roblox instances and leaves layout and design to you.
- State lives in **sources**, reactions in **effects**, and lifetimes in **scopes**.
