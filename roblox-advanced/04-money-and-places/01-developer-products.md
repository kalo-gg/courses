# Developer products

*Bind a receipt handler per product, grant each PurchaseId once, and answer Processed only after the save lands.*

Roblox Advanced · Money and places · 11 min

---

A player buys 100 coins. Your code adds them, the save fails, and the server shuts down: they paid for nothing. Or the save lands but the answer never reaches Roblox, so the same receipt comes back and the coins are granted twice.

Developer products are what a player can buy again and again: currency, boosts, revives. Each purchase arrives as a **receipt**, to be granted exactly once.

## A handler for each product

```luau
local MarketplaceService = game:GetService("MarketplaceService")

local COINS_100 = 1234567 -- the product's ID from the Creator Dashboard

-- show the prompt, for example when the shop's remote fires
MarketplaceService:PromptProductPurchase(player, COINS_100)

local function grantCoins(receipt)
	-- receipt.PurchaseId, receipt.PlayerId, receipt.ProductId, receipt.CurrencySpent
	return Enum.ReceiptDecision.NotProcessedYet -- until the grant is saved
end

MarketplaceService:BindReceiptHandler(Enum.ReceiptType.DeveloperProduct, grantCoins, { COINS_100 })
```

The third argument is the filter: this handler sees only receipts for those product IDs. Without one, a handler is the catch-all for every product no filtered handler claims. Grant only here, never from `PromptProductPurchaseFinished`, which does not mean the purchase went through.

`Processed` means granted **and saved**: the receipt never comes back. Anything less is `NotProcessedYet`: the player left, the profile is not loaded, or the save failed.

So one receipt can reach your handler several times, and the docs add a worse case: a buyer who joins a second server before the first answers can have it running on both at once. The defence is its `PurchaseId`: store it in the buyer's session-locked profile, in the same save as the coins. A handler that finds its id there skips the grant and only saves.

## Grant, record, save, answer

```luau
local Players = game:GetService("Players")

local function grantCoins(receipt)
	local player = Players:GetPlayerByUserId(receipt.PlayerId)
	local profile = player and Profiles.await(player) -- nil if they leave first
	if not profile then
		return Enum.ReceiptDecision.NotProcessedYet
	end

	if not profile.Purchases[receipt.PurchaseId] then
		profile.Coins += 100
		profile.Purchases[receipt.PurchaseId] = true
	end

	if Profiles.save(player) then
		return Enum.ReceiptDecision.Processed
	end
	return Enum.ReceiptDecision.NotProcessedYet
end
```

`Profiles` stands in for a session-locked store like the one you build at the end of Module 3: `await` waits for the load, and `save` returns `true` only when the write landed.

Now the failures. The save fails: the handler answers `NotProcessedYet`, and the redelivery finds the id, saves, and answers `Processed`. A save that errors but landed: same path. Two servers at once: only the one holding the session lock has the profile, so only it can grant.

> **Recognising older code**
>
> `MarketplaceService.ProcessReceipt = function(receiptInfo) … end` is the legacy callback: one per game for every product, answering `Enum.ProductPurchaseDecision.PurchaseGranted` or `NotProcessedYet`. A receipt no bound handler matches still falls through to it, so the two can coexist while a game moves over.

> **One handler per product**
>
> Scribe declares products in its options and grants them through `ProcessReceipt`, answering only after the save confirms; the Scribe course covers it in Production Data. A `BindReceiptHandler` bound for the same product takes precedence, so give each product exactly one handler.

**Key points**

- Bind one handler per product with `BindReceiptHandler(Enum.ReceiptType.DeveloperProduct, handler, { productId })`.
- Answer `Processed` only after the grant is saved; anything else is `NotProcessedYet`, and the receipt comes back later.
- A receipt can arrive more than once, even on two servers at once: store its `PurchaseId` in the session-locked profile with the grant.
- Never grant from `PromptProductPurchaseFinished`; `ProcessReceipt` is the legacy callback.
