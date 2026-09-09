# Context

*Provide a value to a subtree of components without passing it through every props table.*

Vide · Actions, Springs & Context · 9 min

---

Some values are needed everywhere below a certain point: a theme, the current player, a service handle. Passing them through every intermediate component's props is noisy. `context(default)` creates a **context**: call it with a value and a function to provide the value while that function runs, and call it with no arguments from any component created inside to read it.

```luau
local context = vide.context

local Theme = context({ accent = Color3.fromRGB(5, 150, 105) }) -- default

local function Button(props: { Text: string })
	local theme = Theme() -- reads the nearest provided theme
	return create "TextButton" {
		Text = props.Text,
		BackgroundColor3 = theme.accent,
	}
end

local function App()
	return Theme({ accent = Color3.fromRGB(239, 68, 68) }, function()
		-- everything created in here sees the red theme
		return create "ScreenGui" {
			Button { Text = "Danger" },
		}
	end)
end
```

Providing runs the function in a new stable scope and returns whatever it returns.

## Nesting and defaults

Lookup walks up the scope tree, so the nearest provider wins and you can nest providers to override for a subtree. Reading a context that was never provided returns the default (`nil` if you gave none). Providing, on the other hand, must happen inside a scope: outside `root`/`mount` it errors with `attempt to set context outside of a vide scope`.

**Nearest provider wins**

```luau
local Theme = context("light")
print(Theme()) -- "light": no provider, so the default

root(function()
	Theme("dark", function()
		print(Theme()) -- "dark"
		Theme("contrast", function()
			print(Theme()) -- "contrast": the nearest provider wins
		end)
		print(Theme()) -- "dark" again
	end)
end)
```

A context holds a *value*, provided once for the scope. If that value must change over time — the current theme the player can switch — provide a source and read it where needed: `Theme(themeSource, fn)`, then `Theme()()` in a binding. Contexts are for cross-cutting values a subtree needs everywhere; a component's ordinary inputs stay props.

**Key points**

- `context(default)` makes a context; `Ctx()` reads it, `Ctx(value, fn)` provides it while `fn` runs.
- The nearest provider wins; providers nest.
- Use context for cross-cutting values like themes and services, not for ordinary component inputs.
