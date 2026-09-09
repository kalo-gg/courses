# ForValues

*Process every value in a table, keep the keys, and give each entry a scope of its own.*

Fusion · Tables & Lists · 10 min

---

You could write `Computed(function(use) … end)` that loops over a table and builds a new one. It would work, and every change to any entry would rebuild every entry. The `For` family exists to avoid exactly that.

**Doubling every value**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children = Fusion.peek, Fusion.Children

local numbers = scope:Value({ 1, 2, 3 })

local doubled = scope:ForValues(numbers, function(use, scope, num)
	return num * 2
end)

print(table.concat(peek(doubled), ",")) --> 2,4,6

numbers:set({ 1, 2, 3, 4 })
print(table.concat(peek(doubled), ",")) --> 2,4,6,8
```

The source calls it the object that "processes values and preserves keys": the shape stays, the contents change.

## Three arguments, not one

The processor is handed `use` and a `scope` — the same two a `Computed` gets — and then the value. The `use` means an entry can depend on other state, and the `scope` means anything the processor creates has an owner tied to *that entry*, not to the whole list. When the value leaves the table, its scope is cleaned up.

**The processor's own scope**

```luau
local names = scope:Value({ "Ada", "Grace" })

local labels = scope:ForValues(names, function(use, scope, name)
	-- this instance belongs to this entry
	return scope:New "TextLabel" { Text = name }
end)
```

Remove "Ada" from the table and the label made for Ada is destroyed, because it lived in Ada's scope.

> **It reuses what it can — once**
>
> Fusion does not rebuild the output from scratch on every change: it "will try and reuse as much as possible", so a value that merely moves position keeps its processed result. The catch is that each value is only reused once, so a table containing the same value twice recomputes the duplicate.

A `For` object is a state object like any other. `peek` it and you get a plain table; `use` it inside a `Computed` and the computed follows every change to the output; hand it to `[Children]` and you get instances. That is why the family exists as objects rather than as a helper function — the result is something other Fusion objects can depend on.

**Key points**

- `scope:ForValues(input, processor)` processes values and preserves keys.
- The processor is handed `use`, a scope of its own, and the value — in that order.
- Anything the processor creates belongs to that entry's scope and is destroyed when the entry leaves.
- Results are reused one-to-one, so a table with duplicate values recomputes the repeats.
- The result is a state object: `peek` it, `use` it, or drop it into `[Children]`.
