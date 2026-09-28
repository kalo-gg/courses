# Server authority

*The client says what the player wants to do. The server works out what it costs and earns from its own tables, applies it, then tells the clients.*

Roblox Intermediate · Trust · 11 min

---

A chest fires `AddCoins:FireServer(50)` when a player opens it. The handler has every check from this module: the amount must be a finite number from 0 to 100, and there is a rate limit. An exploiter sends 100, five times a second, for as long as they like.

Every check passes, because the server asked the wrong question. It asked whether 50 was a reasonable amount, when the amount was never the client's to say.

## Intent, not outcome

Under **server authority**, the client sends what the player is trying to do: open this chest, buy this sword, fire in this direction. The server then follows the docs' security guide: check that the action is possible and allowed, apply it to its own state, then tell the clients. Prices, damage and rewards come from the server's own tables, so there is no number to forge.

**Server**

```luau
local PRICES = { Sword = 100, Shield = 60 } -- the server's own table

BuyItem.OnServerEvent:Connect(function(player, itemName: unknown)
	local price = if typeof(itemName) == "string" then PRICES[itemName] else nil
	if price == nil then
		return -- not something we sell
	end
	local coins = player.leaderstats.Coins
	if coins.Value < price then
		return
	end
	coins.Value -= price -- 1. apply it on the server
	Purchased:FireClient(player, itemName) -- 2. then tell the client
end)
```

The client names the item. The price, the balance and the decision all come from the server, and the client hears the result.

Once the server has applied a change, the clients hear about it: a leaderstat replicates by itself, and a remote tells a client what happened so it can show it. The client may play an effect at once so the game feels quick, which Module 7 covers, but the server's answer is the one that counts.

**Client**

```luau
-- Ask, then show whatever the server decided
ClaimChest:FireServer(chest)

RewardGranted.OnClientEvent:Connect(function(amount: number)
	rewardLabel.Text = "+" .. amount .. " coins"
end)
```

The client never says how much. It shows the amount the server sent back.

> **Server authority, the engine feature**
>
> Roblox is also building an engine feature called server authority, which moves physics and movement onto the server with client prediction. It is in beta, and this course will teach it once it ships. The design rule in this lesson applies either way. The Blink course's "Validation and the trust boundary" lesson applies the same rule to generated remotes.

**Key points**

- The client sends intent, what the player wants to do, never outcomes such as prices, damage or rewards.
- The server looks those up in its own tables, so there is no number to forge.
- On the server: check that the action is possible and allowed, apply it, then tell the clients.
- A range check on a client-chosen amount still lets an exploiter pick the best value every time.
- Roblox's server authority engine feature is in beta; this design rule applies with or without it.
