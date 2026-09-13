# Container events

*Four events, four questions: something changed, which child moved, a key came or went, an entry was inserted or dropped.*

Scribe · Containers & Big Numbers · 11 min

---

`Changed` on a container answers "something beneath this changed, here is the current state": `(new, old)`, coalesced, and `old` is usually the same table as `new`, because Scribe does not snapshot a container. `OnChildChanged(key, new, old)` answers "which direct child moved": one fire per write. `OnKeyAdded` and `OnKeyRemoved` report a key appearing or disappearing, `(key, element)`; `OnInsert` and `OnRemove` are the array equivalents, `(element, index)`.

**Four listeners on the bag**

```luau
data.Inventory.Changed(function(inventory)
	redrawTotals(inventory) -- once per batch, however many slots moved
end)

data.Inventory.OnChildChanged(function(itemId, new, old)
	refreshSlot(itemId, new) -- once per write
end)

data.Inventory.OnKeyAdded(function(itemId, entry)
	playPickupSound(itemId, entry.Rarity)
end)

data.Inventory.OnKeyRemoved(function(itemId, entry)
	clearSlot(itemId)
end)
```

| Event | Arguments | Answers |
| --- | --- | --- |
| `Changed` | `(new, old)` | Something beneath this changed; here is the current state |
| `OnChildChanged` | `(key, new, old)` | This direct child moved; here are both sides |
| `OnKeyAdded` / `OnKeyRemoved` | `(key, element)` | A key appeared or disappeared |
| `OnInsert` / `OnRemove` | `(element, index)` | An array entry was added or dropped |

## Coalescing and order

A `Data.Batch` writing four entries fires `Changed` once, and on the client one diff frame fires it once however many children it carried. `OnChildChanged` fires per write, and every per-write fire arrives before the container's `Changed`, so you can accumulate keys and act once. A write that replaces the container's own table (a whole-container `Set`, `SetOf` `Add` and `Remove`, `Flags` `Enable`, `Disable` and `Toggle`) carries a real prior table as `old` and is not coalesced.

**Counting fires in one batch**

```luau
local log = {}
data.Inventory.OnKeyAdded(function(key) table.insert(log, "added " .. key) end)
data.Inventory.OnChildChanged(function(key) table.insert(log, "child " .. key) end)
data.Inventory.Changed(function() table.insert(log, "changed") end)

Data.Batch(ava, function()
	data.Inventory.Emberblade.Set({ Qty = 1, Rarity = "Legendary" })
	data.Inventory.Potion.Set({ Qty = 1, Rarity = "Rare" })
end)

print(table.concat(log, " | "))
--> added Emberblade | child Emberblade | added Potion | child Potion | changed
```

## Rules for handlers

The per-write events fire immediately, which means inside any open `Data.Transaction`. A handler that yields rolls the whole transaction back, including the write that triggered it, and the message blames the transaction body, not the listener. A handler that raises is logged and the transaction still commits. Keep listeners synchronous and cheap. In Studio, a container `Changed` listener that declares a third parameter errors at connect time: the `key` argument was removed in 1.3.0.

**Key points**

- `Changed` on a container is state, coalesced, with `old` usually the same table as `new`.
- `OnChildChanged(key, new, old)` fires per write, before the container's `Changed`.
- `OnKeyAdded` and `OnKeyRemoved` report keys appearing and disappearing; `OnInsert` and `OnRemove` do the same for arrays.
- Never yield inside a container listener: it rolls back an open transaction, and the message blames the transaction.
- An eviction is a real removal: it fires `OnRemove` and replicates.
