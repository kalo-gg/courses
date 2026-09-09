# Lists with values

*One scope per distinct value: rows follow their item when the list is reordered.*

Vide · Dynamic Scopes · 10 min

---

`values(list, fn)` mirrors `indexes` with the roles swapped: scopes are keyed by **value**, and `fn(value, index)` receives the value as a plain argument and the *index* as a source. Reordering the table therefore moves rows instead of recreating them, which is what you want when each item has identity (a player, an inventory entry) rather than a slot number.

**Rows that follow their player**

```luau
-- inside a root or component
local values = vide.values

local players = source({ "Ada", "Grace", "Linus" })

create "Frame" {
	create "UIListLayout" {},

	values(players, function(name, i)
		return create "TextLabel" {
			Text = name,               -- plain string, never changes for this row
			LayoutOrder = i,           -- source: updates when the row moves
		}
	end),
}

players({ "Linus", "Ada", "Grace" }) -- same three rows, new LayoutOrders
```

## Identity, not position

Because the scope is keyed by the value, the value can never change for that row, which is why `fn` receives it plainly. What can change is where it sits: reorder the table and each row's index source is written, nothing is rebuilt. A value that leaves the table destroys its scope; a value that arrives creates one. Items with identity — a player, a unique table per entry — are what this is for.

> **Values must be unique**
>
> Because scopes are keyed by value, two equal values in the table would share one scope. In strict mode Vide raises an error for duplicates. If your data can repeat (two players both called "Guest"), use `indexes`, or key rows by a unique table per item.

**Unique tables as values**

```luau
-- names can repeat, so key the rows by a table per player instead of by the string
local players = source({ { name = "Ada" }, { name = "Guest" }, { name = "Guest" } })

values(players, function(player, i)
	return create "TextLabel" {
		Text = player.name,   -- plain: the table never changes for this row
		LayoutOrder = i,      -- source: the row moves when the list is re-sorted
	}
end)
```

## Order comes from LayoutOrder

`values` builds its output array from a map keyed by value, so the order Vide parents the rows in is hash order. The only order a player sees is the one `LayoutOrder` and a layout instance give it.

> **`LayoutOrder` is not decoration — it is the only order you get**
>
> Look at how the output array is built: `for _, scope in scopes do table.insert(objects, scope.object) end`. `scopes` is a **map keyed by value**, so that loop runs in Luau's hash order — arbitrary, and not stable between runs. The array `values()` hands back, and therefore the order the children are parented in, has nothing to do with the order of your list.
>
> So every example here sets `LayoutOrder = i` for a reason. Drive visual order with `LayoutOrder` and a `UIListLayout` (or `UIGridLayout`), and never read `GetChildren()` expecting list order. `indexes` and `switch` build their output arrays exactly the same way.

|  | `indexes` | `values` |
| --- | --- | --- |
| Scope keyed by | index | value |
| `fn` receives | `(valueSource, index)` | `(value, indexSource)` |
| Reordering the table | updates each row's value | moves rows (updates index sources) |
| Duplicates | fine | error in strict mode |
| Good for | positional slots, grids | items with identity, sortable lists |

**Key points**

- `values(list, fn)` keys scopes by value; `fn` gets the value plainly and the index as a source.
- Reordering moves rows instead of rebuilding them.
- Values must be unique; use `indexes` when items can repeat.
