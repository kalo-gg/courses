# Components and props

*Wrap elements in functions, pass props, and compose small pieces into screens.*

Vide · Vide Foundations · 10 min

---

A **component** in Vide is nothing more than a function that returns UI. There is no base class and no registration. Because a component is a plain function, you can call it anywhere, pass it anything, and put its result anywhere an instance goes, including as a child of another `create` call.

**Button.luau**

```luau
-- Button.luau
local create = vide.create

local function Button(props: {
	Position: UDim2,
	Text: string,
	Activated: () -> (),
})
	return create "TextButton" {
		BackgroundColor3 = Color3.fromRGB(50, 50, 50),
		TextColor3 = Color3.fromRGB(255, 255, 255),
		Size = UDim2.fromOffset(200, 50),

		Position = props.Position,
		Text = props.Text,
		Activated = props.Activated,

		create "UICorner" {},
	}
end
```

**Menu.luau**

```luau
-- Menu.luau
local function Menu()
	return create "ScreenGui" {
		Button {
			Position = UDim2.fromOffset(200, 200),
			Text = "back",
			Activated = function() print("go to previous page") end,
		},
		Button {
			Position = UDim2.fromOffset(400, 200),
			Text = "next",
			Activated = function() print("go to next page") end,
		},
	}
end
```

`Button { … }` is a normal function call with a table argument, so it slots in as a child exactly like `create "TextButton" { … }` would.

## Props

By convention a component takes a single `props` table. Typing it (`props: { Text: string }`) gives you autocomplete and catches typos at the call site. Inside, you decide which props map straight onto instance properties, which ones configure behavior, and which have defaults. A component owns its element's look; callers only supply what varies.

Not every prop is required. Give the optional ones a default inside the component with `or`, and mark them optional in the type (`Size: UDim2?`) so the call site can leave them out. Defaults belong in the component, not in every caller.

**Optional props with defaults**

```luau
local function Button(props: { Text: string, Size: UDim2? })
	return create "TextButton" {
		Text = props.Text,
		Size = props.Size or UDim2.fromOffset(200, 50),
	}
end

Button { Text = "OK" }                                  -- default size
Button { Text = "Wide", Size = UDim2.fromOffset(320, 50) }
```

> **Best practice: keep components small and dumb**
>
> A component should build one piece of UI from its props. Data fetching, game logic and network calls belong in the code that *calls* the component. That keeps components reusable and, once sources arrive in Sources, Effects & Scopes, keeps the reactive graph easy to reason about.

**Key points**

- A component is a function that takes `props` and returns an instance.
- Type the props table; put the variable parts in props and the fixed look inside the component.
- Use a component as a child by calling it inside another property table.
- Keep components small and free of game logic; the caller owns behavior.
