# A list that changes

*Drop a For object into Children and the interface follows its data, one instance per entry.*

Fusion · Tables & Lists · 11 min

---

This is where the module pays off. `[Children]` accepts a state object containing instances, and a `For` object is exactly that — so a table of data becomes a row of instances that appears and disappears as the data changes.

**An inventory that updates itself**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children = Fusion.peek, Fusion.Children

local items = scope:Value({ "Sword", "Shield" })

local list = scope:New "Frame" {
	[Children] = scope:ForValues(items, function(use, scope, item)
		return scope:New "TextLabel" { Text = item }
	end),
}

print(#list:GetChildren()) --> 2

items:set({ "Sword", "Shield", "Potion" })
print(#list:GetChildren()) --> 3

items:set({ "Sword" })
print(#list:GetChildren()) --> 1
```

Nothing in this code adds or removes a label. The data changed; the instances followed.

## Where the removed label went

When "Shield" left the table, its label was destroyed — not merely unparented. That is the processor's scope doing its job: the label was created in the scope belonging to that entry, so the entry leaving takes the label with it. This is the same rule from Fusion Foundations, applied one level down.

**A title and a list in one Children array**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children = Fusion.peek, Fusion.Children

local items = scope:Value({ "Sword", "Shield", "Potion" })

local bag = scope:New "Frame" {
	[Children] = {
		scope:New "TextLabel" {
			Text = scope:Computed(function(use)
				return #use(items) .. " items"
			end),
		},

		scope:ForValues(items, function(use, scope, item)
			return scope:New "TextLabel" { Text = item }
		end),
	},
}

print(#bag:GetChildren()) --> 4: the title, and one per item
```

`[Children]` takes an array mixing a plain instance and a `For` object and flattens the lot, so a header and a list sit side by side without a wrapper frame.

> **Do not hold on to instances from a For**
>
> Keeping a reference to a label a `For` made, and using it later, is a bug waiting for the moment its entry leaves the table. Read the list through the `For` object, or through the parent's children.

The rule that follows is worth saying plainly: read a list through its data or through the parent's children, never through a saved reference. `bag:GetChildren()` is always current. A label you stashed in a local three seconds ago may be a destroyed instance whose properties still read back and whose `Parent` is now `nil`.

**Key points**

- A `For` object inside `[Children]` gives you one instance per entry, tracked for you.
- `[Children]` parents and unparents; the entry's scope destroys. Build with the processor's scope.
- An array in `[Children]` may mix plain instances, nested arrays and state objects; it is flattened.
- Never hold on to an instance a `For` produced.
