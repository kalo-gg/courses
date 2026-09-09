# Reactive components

*Combine a source, an effect and an instance into a component that updates itself.*

Vide · Reactive UI · 9 min

---

A reactive component is a component that owns some state and keeps its instances in sync with it. The pieces are exactly the ones from Sources, Effects & Scopes: a source to hold the state, an effect to push it into the instance. Here is the counter from What is Vide? written with an explicit effect.

```luau
local create, source, effect = vide.create, vide.source, vide.effect

local function Counter()
	local count = source(0)

	local button = create "TextButton" {
		Activated = function()
			count(count() + 1)
		end,
	}

	effect(function()
		button.Text = "count: " .. count()
	end)

	return button
end
```

Each call to `Counter()` makes its own `count`, so every counter on screen is independent.

## Who owns the state?

State does not have to live inside the component. Pass a source in through props and the component displays whatever the caller decides. The only rule is the one from Sources, Effects & Scopes: the effect must be created inside a stable scope, which is true whenever the component is called from inside a `root` (or, as you will see in the last lesson of this module, from `mount`).

**State from outside**

```luau
local function CountDisplay(props: { count: () -> number })
	local label = create "TextLabel" {}

	effect(function()
		label.Text = "count: " .. props.count()
	end)

	return label
end

local count = source(0)

root(function()
	CountDisplay { count = count }
end)

count(1) -- the label now reads "count: 1"
```

Which is right? State lives with whoever has to change it. A counter owns its count because nothing else touches it; a health label does not own health, the game does, so it takes the source as a prop. When two components show the same value they must read the same source, never each hold a copy.

**Key points**

- A reactive component pairs a source with an effect that writes to an instance.
- State created inside the component is private to each call; state passed via props is shared.
- Effects inside components need a stable scope around the call, usually a `root` or `mount`.
