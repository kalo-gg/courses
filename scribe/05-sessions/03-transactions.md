# Batch and Transaction

*Batch coalesces, Transaction rolls back, Flush makes it durable; and why one player never needs a protocol.*

Scribe · Sessions & Saving · 11 min

---

Write down the rule you are afraid of breaking as one sentence and count the players' saved data it mentions. One player: `Data.Transaction(player, fn)` makes the writes land together or not at all, and `Data.Flush` makes them durable. The session lock means one server writes a key at a time, and one save writes the whole tree, so there is no second writer to race, which is the problem every distributed commit protocol exists to solve. It does not arise here.

**Emberfall's forge**

```luau
local function forge(player)
	local data = Data[player]

	local ok, err = Data.Transaction(player, function()
		if data.Coins.Get() < 250 then
			error("not enough coins", 0)
		end
		data.Coins.Decrement(250)
		data.Xp.Increment(200)
		data.Inventory.Emberblade.Update(function(entry)
			return { Qty = if entry then entry.Qty + 1 else 1, Rarity = "Rare" }
		end)
	end)

	if not ok then
		return false, err -- nothing changed; the coins are untouched
	end
	Data.Flush(player)
	return true
end
```

|  | `Batch` | `Transaction` |
| --- | --- | --- |
| Coalesces replication | yes | yes |
| Rolls back on error | no | yes |
| Return value | none | `(boolean, string?)` |
| May yield inside | yes | no |

## What atomic does not mean

`Transaction` returning true means the writes landed on the tree together. They queue for the next autosave like any write, and a crash before it loses them: `Flush` is the commit point. Its false is unresolved, not failed, and never a reason to compensate. A transaction refuses to yield (the yield rolls it back, and the message names the transaction body) and refuses to touch another player's tree (the whole transaction fails), because a rollback cannot reach a different DataStore key.

**Rollback, seen twice**

```luau
local ok, err = Data.Transaction(ava, function()
	data.Coins.Set(999)
	error("boom", 0)
end)
print(ok, err, data.Coins.Get()) --> false  boom  100

ok, err = Data.Transaction(ava, function()
	data.Coins.Set(1)
	task.wait() -- a yield rolls the whole transaction back
end)
print(ok, data.Coins.Get()) --> false  100
```

## When the rule names two players

Most cross-player features are one-sided: a gift, mail, a bounty. If the effect on the other player only ever adds, the problem collapses into delivery. Record the debit and the debt in one transaction, `Flush`, then `Data.SendMessage(userId, payload)`, and apply it in `Data.OnMessage` inside a transaction that also writes a marker, so a redelivery does nothing. Delivery is at least once. A trade between two players on one server is `Data.Exchange.Attempt`; across servers there is no API, and the guide explains why.

**Batch, when you only want fewer frames**

```luau
-- Emberfall's respawn: two stats move, the client redraws once.
Data.Batch(player, function()
	local data = Data[player]
	data.Stats.Deaths.Increment(1)
	data.Stats.Playtime.Increment(60)
end)
-- A Transaction already batches, so you never need both.
```

**Key points**

- One player's rule needs no protocol: `Transaction` makes the writes atomic and `Flush` makes them durable.
- A throw or a yield inside the body rolls everything back; a write to another player's tree fails the transaction.
- `Batch` only coalesces replication; a `Transaction` already batches.
- Two players with a monotone far side is a durable outbox; a symmetric trade on one server is `Data.Exchange.Attempt`.
- A false `Flush` is unresolved: never compensate, never re-grant.
