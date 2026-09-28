# Text, chat and safety

*Filter every piece of text a player writes for others, once and on the server, and check what local rules allow each player.*

Roblox Advanced · Running a live game · 10 min

---

A player names their pet, and the name floats above it for everyone in the server. Whatever they typed, a phone number or a slur, every other player now reads.

Roblox filters chat for you. Every other piece of text players write for each other is yours to filter, and the text filtering guide warns that Roblox removes an experience that does not filter until filtering is added.

## Filter once, on the server

```luau
local TextService = game:GetService("TextService")

-- The text everyone in the server may see, or nil when it cannot be filtered
local function filterForEveryone(text: string, author: Player): string?
	local ok, result = pcall(TextService.FilterStringAsync, TextService, text, author.User.Id)
	if not ok then
		return nil -- it already retried: show nothing
	end
	local shown, filtered = pcall(result.GetNonChatStringForBroadcastAsync, result)
	return if shown then filtered else nil
end
```

`FilterStringAsync` takes the author's ID and throws if the author is not in this server. It retries internally, so the docs say not to retry a failure: show nothing instead.

Pick the result call by audience. `GetNonChatStringForBroadcastAsync` is the text everyone in the server may see, such as a sign or a pet's name. `GetNonChatStringForUserAsync(readerId)` filters for one reader, by their age and other details, for a private note. Filter once the player submits, never on each keystroke.

Stored text, such as a pet name saved in a DataStore, is filtered again: save what the player wrote, and filter it each time it is loaded.

> **Recognising older code**
>
> `result:GetChatForUserAsync(userId)` is deprecated and now returns an empty string. Chat goes through `TextChatService`, which filters it; other text uses the two `GetNonChatString…` methods.

## What local rules allow

Some features are allowed for one player and not another. `PolicyService:GetPolicyInfoForPlayerAsync(player)` returns flags worked out from the player's location, age group and platform, such as `ArePaidRandomItemsRestricted` for paid random items, `IsPaidItemTradingAllowed` and `AreAdsAllowed`. A game with that feature must check the flag.

```luau
local PolicyService = game:GetService("PolicyService")

local function canBuyLootBoxes(player: Player): boolean
	local ok, policy = pcall(PolicyService.GetPolicyInfoForPlayerAsync, PolicyService, player)
	-- no answer: assume the stricter rule
	return ok and not policy.ArePaidRandomItemsRestricted
end
```

It is a web request, so it goes in `pcall`. When it fails, this code keeps the feature off for that player.

**Key points**

- Filter text a player writes for others on the server, once per submission, with the author's ID.
- `GetNonChatStringForBroadcastAsync` is for everyone; `GetNonChatStringForUserAsync` is for one reader.
- If filtering fails, show nothing and do not retry: it already did.
- Filter stored text again each time it is loaded. Chat through `TextChatService` is filtered for you.
- Check `PolicyService` flags before showing a feature that local rules may forbid.
