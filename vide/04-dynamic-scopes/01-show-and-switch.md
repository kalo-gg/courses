# show and switch

*Render a component while a condition holds, or pick one of several by a key.*

Vide · Dynamic Scopes · 10 min

---

`show(condition, component)` runs `component` in a fresh stable scope whenever `condition()` becomes truthy and destroys that scope when it becomes falsey. It returns a **source** holding the component's instance (or `nil`; briefly an array of the old and new instances during a delayed destruction), so you can place the call directly among a parent's children.

```luau
local show = vide.show

local function Menu()
	return create "TextLabel" { Text = "This is a menu" }
end

local function App()
	local toggled = source(false)

	return create "ScreenGui" {
		create "TextButton" {
			Text = "Toggle menu",
			Activated = function()
				toggled(not toggled())
			end,
		},

		show(toggled, Menu),
	}
end
```

An optional third argument is a fallback component shown while the condition is falsey.

The component receives two sources: the condition's current truthy value, and `present`, a boolean source that flips to `false` just before the scope is destroyed. You will use `present` for exit animations in the last lesson of this module.

## switch

`switch(key)(map)` generalises `show` to many cases: `map` pairs values with component constructors, and whichever one matches `key()` is run in its own stable scope. Changing the key destroys the old scope and creates the new one. A key with no entry shows nothing.

```luau
-- inside a root or component
local switch = vide.switch

local page = source("home")

create "Frame" {
	switch(page) {
		home = HomePage,
		shop = ShopPage,
		settings = SettingsPage,
	},
}

page("shop") -- HomePage's scope is destroyed, ShopPage's is created
```

**Key points**

- `show(condition, component, fallback?)` runs the component in a stable scope while the condition is truthy.
- `switch(key)(map)` picks one component by value and swaps scopes when the key changes.
- Both return a source of instances, so they slot straight in as children.
