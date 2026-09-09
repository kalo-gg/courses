# Creating UI with create

*Build instances with create, set properties, add children and connect events from one table.*

Vide · Vide Foundations · 10 min

---

Everything in Vide starts with `create`. Give it a class name and it returns a **constructor**: a function that takes a table of properties and returns a new instance. Luau lets you drop the parentheses when the only argument is a string or a table, which is what makes Vide code read like a layout.

**One element, then a tree**

```luau
local create = vide.create

local label = create "TextLabel" {
	Text = "hi",
	Size = UDim2.fromOffset(200, 40),
}

local gui = create "ScreenGui" {
	create "Frame" {
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.5, 0.5),
		Size = UDim2.fromScale(0.4, 0.7),

		create "TextLabel" { Text = "hi" },
		create "TextLabel" { Text = "bye" },

		create "TextButton" {
			Text = "click me",
			Activated = function()
				print("clicked!")
			end,
		},
	},
}
```

`create "TextLabel" { … }` is the same as `create("TextLabel")({ … })`.

## The rules of the property table

| Key | Value | Vide does |
| --- | --- | --- |
| String (`Text`) | A plain value | Sets the property |
| String (`Activated`) | A function, and the property is an event | Connects the function to the event |
| String (`Text`) | A function, and the property is not an event | Creates an effect that keeps the property in sync (Reactive UI) |
| Number (`[1]`) | An instance | Parents it as a child |
| Number | A table | Treated as a nested property table: instances inside are parented, keyed entries set properties |
| Number | A function | Binds children reactively (Reactive UI) |
| Number | An action | Runs it on the instance (Actions, Springs & Context) |

Children are just values without a key, so `create "Frame" { create "UICorner" {} }` is valid and reads naturally. `Parent` is special: Vide sets it **last**, after every other property and child, so the instance appears in the world fully built.

> **Sensible defaults**
>
> With `vide.defaults` on (the default), `create` tidies up Roblox's defaults for you: frames get `BorderSizePixel = 0`, buttons get `AutoButtonColor = false`, `ScreenGui`s get `ResetOnSpawn = false`, list layouts sort by `LayoutOrder`. Set `vide.defaults = false` if you prefer the raw engine defaults.

> **0.4.1's typed class list is short**
>
> `create` is typed against a hand-written list of 19 classes in `create.luau`: `Folder`, `BillboardGui`, `CanvasGroup`, `Frame`, `ImageButton`, `ImageLabel`, `ScreenGui`, `ScrollingFrame`, `SurfaceGui`, `TextBox`, `TextButton`, `TextLabel`, `UIAspectRatioConstraint`, `UICorner`, `UIGradient`, `UIGridLayout`, `UIListLayout`, `Camera` and `WorldModel`.
>
> Anything else — `UIPadding`, `UIStroke`, `UIScale`, `ViewportFrame`, `ProximityPrompt` — **runs fine but does not typecheck**, because there is no entry to look the class up in. Cast the result, or use `Instance.new` for that one element, until a release after 0.4.1 lands the longer list. Every class this course uses is on the list.

## Events

When the key names an event such as `Activated`, `MouseEnter` or `FocusLost`, a function value is **connected** to it rather than bound as a property. Vide connects events after the properties are set, so a handler never runs against a half-built instance. In the Kalo sandbox, `click(button)` fires a button's `Activated` and `MouseButton1Click` events so you can test handlers.

**Events are keys too**

```luau
create "TextButton" {
	Text = "Buy",
	Activated = function()
		print("bought")
	end,
	MouseEnter = function()
		print("hovering")
	end,
}
```

**Key points**

- `create "Class" { … }` returns a new instance; the table mixes properties, children and events.
- String keys set properties (or connect events); unkeyed values are children.
- `Parent` is applied last, after everything else is in place.
- Vide's defaults tidy common properties like `BorderSizePixel` and `AutoButtonColor`.
