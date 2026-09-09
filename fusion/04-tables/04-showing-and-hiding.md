# Showing and hiding

*Two ways to make a panel come and go: rebuild it from a Computed, or leave it there and bind Visible.*

Fusion · Tables & Lists · 10 min

---

A list is not the only thing that comes and goes. An error banner, an empty-bag notice, a details pane that only exists once something is selected: each is a child that is sometimes there and sometimes not. Fusion gives you two answers, and they are not interchangeable.

**A child that is sometimes nothing**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children = Fusion.peek, Fusion.Children

local warning = scope:Value(nil)

local panel = scope:New "Frame" {
	[Children] = scope:Computed(function(use, scope)
		local message = use(warning)
		if message == nil then
			return nil -- no child at all
		end
		return scope:New "TextLabel" { Name = "Warning", Text = message }
	end),
}

print(#panel:GetChildren())          --> 0
warning:set("Not enough coins")
print(panel.Warning.Text)            --> Not enough coins
warning:set(nil)
print(#panel:GetChildren())          --> 0
```

`[Children]` reads a state object and treats `nil` as the absence of a child — the source says so in a comment. Build the label with the callback's own `scope` and it is destroyed on the way out.

## Or leave it there

The other answer is to build the child once and bind `Visible` to a `Computed` boolean. The instance never goes away, so it costs memory whether or not anyone can see it — but it keeps everything it was holding, it can animate out instead of vanishing, and there is nothing to rebuild when it comes back.

**Visible follows a Computed**

```luau
local selected = scope:Value(nil)

local details = scope:New "Frame" {
	Name = "Details",
	Visible = scope:Computed(function(use)
		return use(selected) ~= nil
	end),
	[Children] = scope:New "TextLabel" {
		Text = scope:Computed(function(use)
			return use(selected) or "nothing selected"
		end),
	},
}

print(details.Visible)   --> false
selected:set("Sword")
print(details.Visible)   --> true
```

One instance, two computeds. The frame is always a child of whatever parents it; only `Visible` moves.

|  | Rebuild from a Computed | Bind Visible |
| --- | --- | --- |
| Instance while hidden | Does not exist | Exists, hidden |
| State inside it | Lost on every change | Kept |
| Cost of showing again | Rebuild the subtree | One property write |
| Can animate out | No — it is gone before the animation runs | Yes: fade or slide, then hide |
| Good for | Content that differs every time: an error message, the selected item's details | A fixed panel that toggles: a menu, a tooltip, a confirmation |

Both shapes end up in the same `[Children]` array as your lists, because `[Children]` flattens: a header instance, a `ForValues` of rows and a `Computed` that is usually `nil` can all sit in one table. That is the whole vocabulary of a Fusion screen — a static part, a repeated part, and a conditional part.

**Key points**

- A state object holding `nil` inside `[Children]` means "no child here".
- A `Computed` that returns an instance rebuilds it on every change; the callback scope destroys the old one.
- Binding `Visible` keeps the instance and its state, and is what lets it animate out.
- Rebuild when the contents differ each time; keep and hide when the panel is fixed.
- Static, repeated and conditional children all live in the same flattened `[Children]` array.
