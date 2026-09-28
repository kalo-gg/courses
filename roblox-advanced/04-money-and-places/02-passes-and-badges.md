# Game passes and badges

*Ask for pass ownership on join, grant from the purchase event, and award badges without letting one failed call stop the rest.*

Roblox Advanced · Money and places · 10 min

---

A player buys your VIP pass in the shop, and nothing happens until they rejoin. Another owns VIP, joins during a hiccup, and the error from the ownership check ends their join handler, welcome badge and all.

Passes and badges are owned once, and Roblox keeps the record, so there is no receipt to handle. Your job is to ask at the right moments, and to survive the asking.

## Passes: on join, and on purchase

```luau
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")

local VIP_PASS = 777 -- a pass ID, not an asset ID

local function giveVip(player: Player)
	player:SetAttribute("VIP", true)
end

Players.PlayerAdded:Connect(function(player)
	local ok, owns = pcall(MarketplaceService.UserOwnsGamePassAsync, MarketplaceService, player.User.Id, VIP_PASS)
	if ok and owns then
		giveVip(player)
	end
end)

MarketplaceService.PromptGamePassPurchaseFinished:Connect(function(player, passId, wasPurchased)
	if wasPurchased and passId == VIP_PASS then
		giveVip(player)
	end
end)
```

The ownership check is a web call, so it goes in `pcall`. The purchase event already says what was bought, so the second handler grants without asking again.

`PromptGamePassPurchaseFinished` fires when the pass prompt closes, whether the player bought, cancelled or hit an error, and for every pass you sell. So check both `wasPurchased` and the ID. It also refreshes the engine's cache, so a later `UserOwnsGamePassAsync` agrees with it.

## Badges

```luau
local BadgeService = game:GetService("BadgeService")

local WELCOME_BADGE = 2124000000

local function awardWelcome(player: Player)
	local ok, result = pcall(BadgeService.AwardBadgeAsync, BadgeService, player.User.Id, WELCOME_BADGE)
	if not ok then
		warn("Welcome badge failed:", result)
	end
end
```

`AwardBadgeAsync` returns `true` when it awarded the badge. A player who already owns it cannot be awarded it again, unless they deleted it from their profile. `UserHasBadgeAsync` answers whether they own it.

An award succeeds only from a server script, for a player who is in the experience, for an enabled badge that belongs to this experience. Like every call here it can fail, so give each web call its own `pcall`: a failed badge must not cost the player their VIP, and the other way round.

| API | What it does |
| --- | --- |
| `PromptSubscriptionPurchase(player, subscriptionId)` | Shows the subscription prompt |
| `GetUserSubscriptionStatusAsync(player, subscriptionId)` | Returns `{ IsSubscribed, IsRenewing }`; `IsSubscribed` is true only once payment has gone through |
| `Players.UserSubscriptionStatusChanged(player, subscriptionId)` | Fires on the server when a status changes, checked after a subscription prompt closes |
| `GetProductInfoAsync(id, Enum.InfoType.Product)` | Name, price and more for a product; `GamePass` and `Subscription` work too, and it throws for an unknown ID |

A subscription can lapse, so check it like a pass and listen for the change. All but the event are on `MarketplaceService`.

> **Recognising older code**
>
> `BadgeService:AwardBadge`, `UserHasBadge` and `MarketplaceService:GetProductInfo` are deprecated: write the `Async` versions, which take the same arguments. `PlayerOwnsAsset` with a pass's asset ID is older still; passes have their own IDs and `UserOwnsGamePassAsync`.

**Key points**

- Check a pass with `UserOwnsGamePassAsync` on join, and grant a purchase made here from `PromptGamePassPurchaseFinished`, checking `wasPurchased` and the ID.
- Ownership is cached: a pass bought outside the experience mid-session can take minutes to show.
- `AwardBadgeAsync` works only on the server, for a player in the experience, and a second award of an owned badge does not succeed.
- Give every web call its own `pcall`, so one failure cannot stop the rest of a join.
