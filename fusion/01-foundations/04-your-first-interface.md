# Your first interface

*New, Children and OnEvent: a counter button that builds itself and cleans itself up.*

Fusion · Fusion Foundations · 11 min

---

`New` is curried: it takes the scope and a class name, and hands back a function that takes the property table. That is why it reads as `scope:New "TextButton" { … }` with no comma between the class and the braces — Luau lets you call a function with a single string or table argument without parentheses.

**A counter**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)

local count = scope:Value(0)

local gui = scope:New "ScreenGui" {
	Parent = game:GetService("Players").LocalPlayer.PlayerGui,

	[Fusion.Children] = scope:New "TextButton" {
		Size = UDim2.fromOffset(200, 50),

		Text = scope:Computed(function(use)
			return "Clicked " .. use(count) .. " times"
		end),

		[Fusion.OnEvent "Activated"] = function()
			count:set(Fusion.peek(count) + 1)
		end,
	},
}
```

Two special keys: `[Children]` parents instances, and `[OnEvent "Activated"]` connects an event whose connection belongs to the scope.

Read the button's `Text` again. It is not a string — it is a `Computed`, and Fusion knows how to unwrap one into a property. Anywhere a property takes a value, it will also take a state object, which is the type Fusion calls `UsedAs<T>`: "a `T`, or something that produces a `T`".

> **peek to write, use to read**
>
> Inside the click handler the code uses `peek(count)`, not `use(count)`. `use` only exists inside a Fusion callback — a `Computed`, or a `For` processor, and it would be wrong here anyway: incrementing a counter should not make the handler depend on the counter. `peek` reads the current value and records nothing.

## Taking it down again

One line tidies all of it away.

**Cleaning the whole screen up**

```luau
print(gui.Parent)                 --> the PlayerGui folder
print(#scope)                     --> 7: the Value, the Computed, the two instances,
                                  --  the Activated connection, the Text binding and the Children processor

Fusion.doCleanup(scope)

print(gui.Parent)                 --> nil
print(button.Parent)              --> nil: the child went with it
```

Seven entries, one call, reverse order — every binding Fusion made on your behalf is in the array too. Nothing in the counter had to know it was being torn down.

`Fusion.doCleanup(scope)` destroys the `ScreenGui`, disconnects the `Activated` connection and drops the `Value` and the `Computed`, in reverse order, without you listing a single one of them.

**What you can do now**

- Install Fusion 0.3 and confirm it with `Fusion.version`.
- Recognize 0.2 snippets by their missing scope argument.
- Create a scope with `scoped(Fusion)` and destroy it with `doCleanup`.
- Read state with `use` inside a Computed and with `peek` everywhere else.
- Build instances with `New`, parent them with `[Children]` and handle input with `[OnEvent]`.
