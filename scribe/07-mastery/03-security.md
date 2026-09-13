# The security line

*A client asks and the server decides; what Scribe checks before your code runs, and which decisions stay yours.*

Scribe · Shipping Emberfall · 10 min

---

A client cannot write player data. Not "writes are validated": there is no frame for it. A client emits exactly two frame types, `Hello` and `Command`, and the server's dispatcher errors on anything else, so no crafted traffic can say "set my Coins to 999". Client-side `Set` touches the local mirror, and the next diff overwrites it. The one shape a safe feature takes: a command whose `Args` prove the shape, and a handler that proves the entitlement.

**The potion shop, both checks named**

```luau
Data.Command("BuyPotion", {
	Args = { Scribe.Int(1, { Min = 1, Max = 10 }) }, -- Scribe proves: an integer, 1 to 10
}, function(player, count)
	return Data.Purchase(player, { -- you prove: they can afford it
		Cost = { Path = "Coins", Amount = 25 * count },
		Category = "Item",
		ItemId = "Potion",
		Grant = function(data)
			if data.Inventory.Potion.Get() == nil then
				data.Inventory.Potion.Set({ Qty = count, Rarity = "Common" })
			else
				data.Inventory.Potion.Qty.Increment(count)
			end
		end,
	})
end)
-- Purchase refuses if the player cannot afford it and rolls back if the grant throws.
-- Scribe never does the second check for you.
```

## The inbound gauntlet

Every frame passes the same gates, cheap ones first: a session check; a malformed budget and an oversize budget of 32 per session, past which that player's frames are dropped unparsed; `MaxInboundFrameRate`, which drops silently; `MaxInboundBytes` before decode; `MaxInboundRetainedBytes` for what decoding would allocate; a recursion cap; the protocol version and the schema hash, both fail closed; a re-`Hello` cooldown. Each logs at most once per second per player, so a flood cannot evict the diagnostic ring.

| Guard | Default | What it stops |
| --- | --- | --- |
| Session check | always | frames from a player with no session |
| Malformed / oversize budgets | 32 each, per session | a client that is not Scribe's; frames dropped unparsed after |
| `MaxInboundFrameRate` | `max(60, CommandRateLimit * 4)` | a flood; drops silently |
| `MaxInboundBytes` | 8192 | an oversized payload, before decode |
| `MaxInboundRetainedBytes` | per options | a payload that decodes into too much memory |
| Recursion cap | 24 | deeply nested arguments |
| Protocol version and schema hash | fail closed | a client compiled against another template |

## What it does not do

No enforcement: crossing a budget stops parsing, nothing kicks or bans. No anti-cheat, no ban list, no per-viewer visibility (`Shared` is all or nothing), no sandbox against your own server code. `OnAnomaly` is a correctness signal: every reason it fires with comes from a server-side write, so building player-facing enforcement on it kicks people for your own bugs. A derived field is the one sanctioned leak: `Watchlisted = suspicion >= 80` leaks a bit, `math.floor(suspicion / 10)` leaks the value; bucket deliberately.

**Gate on the server, never the mirror**

```luau
-- Server: the check that counts.
if Data.OwnsAsync(player, "VIP") then
	Data[player].Gems.Increment(100)
end

-- Client: a mirror can be made to say anything. Use it for the UI, never for a grant.
if Client.Owns("VIP") then
	vipBadge.Visible = true
end
```

> **What to alert on**
>
> `MALFORMED_FRAME_LIMIT` and `INBOUND_OVERSIZE_LIMIT` (not a Scribe client), `INBOUND_RATE_LIMITED`, `COMMAND_UNKNOWN` in volume (probing), `PURCHASE_ID_EVICTED` (a retried receipt could double-grant), `WIPE_GUARD_TRIPPED`; and `ANOMALY` as a bug report to yourself.

**Key points**

- A client emits two frame types; there is no write frame, so the mirror is only a mirror.
- `Args` prove the shape before your handler runs; the handler proves the entitlement.
- Every inbound frame passes budgets, rate and size limits, and a schema hash that fails closed.
- Scribe enforces nothing: no kicks, no bans, no anti-cheat, no per-viewer visibility.
- `ANOMALY` is your bug report; the transport codes are the exploit signals.
