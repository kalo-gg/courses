# Lists with indexes

*One scope per table index: rows are reused when values change and destroyed when indexes vanish.*

Vide · Dynamic Scopes · 10 min

---

`indexes(list, fn)` builds one component per **index** of the table in `list()`. For each index it creates a stable scope and calls `fn(value, index)`, where `value` is a *source* of that slot's current value. When the table is written again, Vide compares indexes: new ones get scopes, missing ones are destroyed, and existing ones just have their `value` source updated, so the row instance is reused rather than rebuilt.

**The row: a value source and a position**

```luau
local indexes = vide.indexes

local function Todo(props: { Text: () -> string, Position: number, Activated: () -> () })
	return create "TextButton" {
		Text = function()
			return props.Position .. ": " .. props.Text()
		end,
		LayoutOrder = props.Position,
		Activated = props.Activated,
	}
end
```

**The list**

```luau
local function TodoList(props: { List: () -> { string } })
	return create "Frame" {
		create "UIListLayout" {},

		indexes(props.List, function(text, i)
			return Todo {
				Text = text,
				Position = i,
				Activated = function() -- remove this todo
					local list = props.List()
					table.remove(list, i)
					props.List(list)
				end,
			}
		end),
	}
end
```

`text` is a source, so binding `Text` to it keeps the row current if that slot's string changes.

The row receives its slot as a *source*. Bind to it (`Text = function() return name() end`, or `Text = name` since a source is a function) rather than copying it once: when the slot's value changes, the same row updates instead of being rebuilt.

## What a write does

When the list source is written, Vide reads the new table and walks its indexes in order: an index it has not seen gets a stable scope and a call to `fn`; an index it already has keeps its scope and, if the value differs, has its `value` source written. Then every scope whose index is missing is destroyed (after its delay, if the component asked for one), and only if a scope was created or destroyed is the output array rebuilt. That array is collected from a map, so its order is not your list's order — set `LayoutOrder` and let a layout instance sort, and never read `GetChildren()` expecting list order.

> **Tables in slots**
>
> A slot's `value` source only fires when the value stored at that index is a different value (`~=`). If the items are tables and you change a field inside one, the slot still holds the same table and nothing updates. Store a replacement table at that index instead.

> **Write the table back**
>
> `table.insert(list, x)` mutates the table but tells no one. Effects and `indexes` only react when you write the source: `list(list())` or, more explicitly, `local t = list(); table.insert(t, x); list(t)`. Vide treats writing the same table as a change for exactly this reason.

**Key points**

- `indexes(list, fn)` creates one stable scope per index; `fn` receives the value as a source and the index as a plain value.
- Existing indexes are updated in place; only new indexes create scopes and only removed ones destroy them.
- Mutating a table does nothing until you write the source again.
