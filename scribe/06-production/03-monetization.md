# Products, passes and purchases

*Declare products and passes next to the template, check what a player owns on the server, and spend soft currency as one write.*

Scribe · Production Data · 11 min

---

`Products` and `Passes` live in the options, named, each with an `Id`; a product's `Grant` receives the buyer's accessor. With any product declared, Scribe takes over `ProcessReceipt`, grants, logs, and answers `PurchaseGranted` only after the save confirms; anything else is `NotProcessedYet`, so Roblox retries and a player never pays for nothing. A `Grant` runs inside a transaction and must not yield. `Data.PromptPurchase(player, name)` prompts by name and refuses something already owned. The sandbox has no prompt, so here it answers `false`; the receipt path is exercised with `Data.TryHandleReceipt(receipt)`, the call a game with its own `ProcessReceipt` makes.

**Two products and a pass**

```luau
Products = {
	CoinPack500 = {
		Id = 1234567890,
		Category = "Currency",
		Grant = function(data)
			data.Coins.Increment(500, { Source = "robux" })
		end,
	},
	GemPack100 = {
		Id = 1234567891,
		Category = "Currency",
		Grant = function(data)
			data.Gems.Increment(100, { Source = "robux" })
		end,
	},
},
Passes = { VIP = { Id = 987654321 } },
Perks = { "VIP" },
```

## Ownership

Perks, passes and Roblox Premium answer to one key: `Owns(player, "VIP")` reads the cache without yielding, `OwnsAsync` re-checks live when the cache does not say owned, `GrantPerk`/`RevokePerk` set a saved boolean by hand, and `OnOwnershipChanged` fires `(player, key, owned)`. Gate a grant on the server's check, never the client's mirror. `Owns` can read false for a moment after join while the refresh runs. `"RobloxPlus"` is built in.

**Grant, observe, revoke, and a receipt by hand**

```luau
print(Data.Owns(ava, "VIP")) --> false
Data.OnOwnershipChanged:Connect(function(player, key, owned)
	print(player.Name, key, owned)
end)
Data.GrantPerk(ava, "VIP") --> Ava VIP true
print(Data.GetProductState(ava, "VIP")) --> owned
Data.RevokePerk(ava, "VIP") --> Ava VIP false

-- The table Roblox hands ProcessReceipt, which is why a game with its
-- own handler can route Scribe's products through it.
local decision = Data.TryHandleReceipt({
	PlayerId = 101,
	ProductId = 1234567890,
	PurchaseId = "r-1",
	CurrencySpent = 99,
	PlaceIdWherePurchased = game.PlaceId,
})
print(decision, data.Coins.Get()) --> Enum.ProductPurchaseDecision.PurchaseGranted  600
```

## Soft-currency purchases

`Data.Purchase(player, spec)` is the shop-side twin: it debits `Cost.Path` by `Cost.Amount`, runs `Grant` and logs it, all or nothing; `insufficient funds` changes nothing, and the refusals are on `Scribe.PurchaseReason`. An `IdempotencyKey` makes a double click free: a repeat under the same key returns the first result and spends nothing. `GetPurchases` reads both logs newest first.

**Buy a lantern, once**

```luau
local function buy()
	return Data.Purchase(ava, {
		Cost = { Path = "Coins", Amount = 30 },
		Category = "Item",
		ItemId = "EmberLantern",
		IdempotencyKey = "buy:1",
		Grant = function(d)
			d.Inventory.EmberLantern.Set({ Qty = 1, Rarity = "Rare" })
		end,
	})
end

print(buy(), data.Coins.Get()) --> true  70
print(buy(), data.Coins.Get()) --> true  70   (the first result again; nothing spent)

data.Coins.Set(10)
print(Data.Purchase(ava, { Cost = { Path = "Coins", Amount = 9999 }, Category = "Item", ItemId = "Castle" }))
--> false  insufficient funds
print(#Data.GetPurchases(ava, { Kind = "InGame" })) --> 1
```

> **A grant through an unresolved key takes the money and delivers nothing**
>
> `data.Inventory[itemId].Qty.Increment(1)` never errors: a new key is created with defaults. Check `data.Inventory[itemId].Qty.Get() == nil` first and `error` inside the grant, which rolls the debit back.

**Key points**

- Products and passes are named in the options; a `Grant` gets the buyer's accessor and must not yield.
- `PurchaseGranted` is answered only after the save confirms; everything else is `NotProcessedYet` and Roblox retries.
- `Owns` reads a cache, `OwnsAsync` verifies, `GrantPerk` is the manual switch; gate on the server.
- `Data.Purchase` debits, grants and logs as one write; an `IdempotencyKey` makes a repeat free.
- Resolve an item id before a grant writes through it: an unresolved key silently creates an entry.
