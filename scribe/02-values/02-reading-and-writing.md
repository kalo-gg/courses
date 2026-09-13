# Reading and writing values

*Get, Set, Update, Clone and Default cover every field; numbers and booleans add a few more, and Get on a table has a trap.*

Scribe · Fields & Values · 10 min

---

You reach a field by indexing the accessor tree the way you wrote the template, and every read and write goes through a method on it. `Get` and `Set` cover most of what you will write; `Increment` and `Decrement` return the new value. Every write is validated, clamped if out of bounds, replicated to the owner, and announced to any listener.

**The common case**

```luau
local data = Data.WaitForData(player)
if not data then
	return
end

data.Coins.Get() --> 0
data.Coins.Set(100) --> 100, the stored value
data.Coins.Increment(50) --> 150
data.Coins.Decrement(25) --> 125

data.Stats.Deaths.Increment(1) -- a nested field is reached the same way
```

## Read, modify, write

`Update(fn)` reads the current value, hands it to your function and stores what it returns. `Clone()` is a deep copy you may edit; nothing is written back. `Default()` is the declared default, read-only schema metadata that works on the client and before any data has loaded, so a "reset to defaults" button reads it rather than repeating the template. `Min()` and `Max()` are the same for numbers, nil where no bound was declared.

**Update, Clone, Default**

```luau
-- Round Emberfall's coin balance down to the nearest ten.
data.Coins.Update(function(current)
	return current - (current % 10)
end)

local snapshot = data.Stats.Clone() -- yours to edit; nothing is written back
snapshot.Deaths = 0

data.Nick.Set(data.Nick.Default()) -- back to the template default

levelBar.Size = UDim2.fromScale(Data.Level.Get() / Data.Level.Max(), 1) -- on the client
```

> **Get on a table hands you the stored table**
>
> For a table field, `Get` returns the stored table itself, not a copy. Mutating it writes straight into authoritative state behind Scribe's back: no validation, no replication, no `Changed`, and the value even saves. Write through the accessor at that key (`data.Inventory[itemId].Rarity.Set(…)`), or edit a `Clone`. Inside `Update`, build a new table and return it. The one read that does not hand back the store is `Data.Get(player).Get()` with no field, which returns a frozen table so the mistake fails loudly.

## Numbers, booleans, and one write off the wire

`Increment` and `Decrement` take an optional second argument, an economy tag table (`TransactionType`, `ItemSku`), which puts the write on Roblox's economy dashboard with the field's name as the currency. Pass `false` there instead and the write is not replicated: it still saves, the client is simply not told, which suits a field that changes every second and no UI reads.

A plain boolean has `Toggle()`, which flips it and returns the new value.

**Tags, silence and a toggle**

```luau
data.Coins.Increment(500, {
	TransactionType = Enum.AnalyticsEconomyTransactionType.IAP,
	ItemSku = "CoinPack500",
})

data.Stats.Playtime.Increment(60, false) -- saved, never sent to the client

local muted = data.Muted.Toggle() --> true, then false next time
```

**Key points**

- `Get`, `Set`, `Update`, `Clone` and `Default` exist on every field; `Set` returns what was stored.
- `Default`, `Min` and `Max` read the schema, so they work on the client and before data loads.
- `Get` on a table is the stored table: never mutate it; write through the accessor or edit a `Clone`.
- `Increment(n, tags)` reports to the economy dashboard; `Increment(n, false)` keeps the write off the wire.
- A plain boolean has `Toggle()`, which returns the new state.
