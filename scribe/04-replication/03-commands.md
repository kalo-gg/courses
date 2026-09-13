# Commands

*Register a handler on the server, call it by name from the client, and let an Args spec prove the shape before your code runs.*

Scribe · Replication & Commands · 11 min

---

A client cannot change data; a command is how it asks. `Data.Command(name, spec?, handler)` registers once at startup; `Data.Request(name, …)` on the client yields until the reply and hands back whatever the handler returned. The `player` your handler receives comes from the transport, so a client cannot claim to be someone else: never accept a user id as an argument and act on it. An unregistered name answers `unknown-command`; registering twice throws.

**ToggleMusic, both ends**

```luau
-- server
Data.Command("ToggleMusic", { Args = { "boolean" } }, function(player, on)
	if on then
		Data[player].Settings.Enable("Music")
	else
		Data[player].Settings.Disable("Music")
	end
	return on
end)

-- client
local musicOn = Data.Request("ToggleMusic", false) --> false
```

## Validating what the client sent

`Args` is a list of specs, one per positional argument, checked before the handler runs, and it checks shape only; whether this player owns that item stays your job. A bad argument is rejected with `bad-args`, never clamped: clamping is how "buy 999999" becomes "buy 10" and a player is charged for what they did not ask. Non-finite numbers are refused whatever the spec; a nil in the list is refused at registration.

| Spec entry | Accepts |
| --- | --- |
| `"string"`, `"number"`, `"boolean"`, `"table"`, `"buffer"` | a value whose typeof is exactly that |
| `"any"` | anything but nil |
| `"string?"` | that type or nil, an omitted trailing argument included |
| `"any?"` | anything at all |
| a declarator, `Scribe.Int(1, { Min = 1, Max = 10 })` | its type and its constraints |
| `{ Field = spec, … }` | a table with exactly these keys |
| `{ spec }` | an array whose every element matches |

**Ranges, and a shape**

```luau
Data.Command("BuyPotion", {
	Args = {
		Scribe.Enum("Health", { "Health", "Mana" }),
		Scribe.Int(1, { Min = 1, Max = 10 }),
	},
}, function(player, flavour, count)
	-- flavour is one of the two; count is an integer from 1 to 10. Both proven already.
end)

Data.Command("CraftItem", {
	Args = { { ItemId = "string", Count = "number?", Reagents = { "string" } } },
}, function(player, order)
	-- order.ItemId is a string, order.Count a number or nil, every reagent a string
end)
```

## Handlers

A handler runs inside an `xpcall` and may yield: a DataStore call, an ownership check. Its return values reach the caller verbatim, at most eight. The client is the side with a clock: `RequestTimeout`, ten seconds by default, resolves the caller with `false, "timeout"` and the late reply is discarded; a timeout is not a rollback, so keep handlers inside it. Inside a handler `Data[player]` is safe without `WaitForData`, because the ready gate ran first.

**A shop command and its replies**

```luau
-- server
Data.Command("Buy", { Args = { "string" } }, function(player, itemId)
	local price = PRICES[itemId]
	if not price then
		return false, "no such item"
	end
	local data = Data[player]
	if data.Coins.Get() < price then
		return false, "too poor"
	end
	data.Coins.Decrement(price)
	data.Inventory[itemId].Set({ Qty = 1 })
	return true, data.Coins.Get()
end)

-- client
print(Client.Request("Buy", "Torch")) --> true  90
print(Client.Request("Buy", "Emberblade")) --> false  too poor
print(Client.Request("Buy", "Nothing")) --> false  no such item
```

**Key points**

- `Data.Command(name, spec?, handler)` on the server; `Data.Request(name, …)` on the client, which yields for the reply.
- The player comes from the transport; never trust a user id argument.
- `Args` proves shape: types, `?` for optional, declarators for ranges, shape tables for payloads.
- A bad argument is rejected with bad-args, never clamped.
- Handlers may yield; the client's `RequestTimeout` is the only clock, and a timeout is not a rollback.
