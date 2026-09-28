# Never trust the client

*An exploiter controls their own client completely, and nothing else. Know where that line runs before you write a handler.*

Roblox Intermediate · Trust · 10 min

---

A shop's Buy button runs `BuyItem:FireServer("Sword", 100)`, sending the item and its price. A week after launch, players own swords they never paid for.

Nobody clicked the button. They ran their own code on their own device and fired `BuyItem` with a price of 0. This lesson maps what such a player, an **exploiter**, controls and what they cannot reach.

## Their client is theirs

Roblox's security guide starts from one rule: never trust the client. An exploiter can run any code on their device, so they can:

- fire any remote, as often as they like, with any arguments
- decompile every LocalScript and ModuleScript that replicated to them, even ones that never run there
- move their own character, and unanchored parts the server lets them own
- fire `ClickDetector` events from any distance, and `ProximityPrompt` events even while the prompt is disabled

**Code an exploiter runs on their own client**

```luau
local BuyItem = game:GetService("ReplicatedStorage"):WaitForChild("BuyItem")

for _ = 1, 1000 do
	BuyItem:FireServer("Sword", 0)
end
```

Ordinary client code. A cooldown in your LocalScript, a hidden button or a price check on the client all run on the exploiter's machine, so they can skip every one of them.

## What stays out of reach

The server is the one machine the exploiter cannot touch. ServerScriptService and ServerStorage never replicate, so they cannot read them. Their changes to the world stay on their own screen: editing their Coins or deleting a wall changes only their copy.

In a remote handler, the engine adds the first parameter, so `player` is always the real sender.

**Server**

```luau
BuyItem.OnServerEvent:Connect(function(player, itemName, price)
	-- player: added by the engine, so it is always the sender
	-- itemName, price: chosen by the client, so check them or ignore them
end)
```

The rest of this module is about those other arguments: checking their types and values, limiting how often they arrive, and not asking for the ones the server can work out itself.

> **Prompts and click detectors count too**
>
> They are not remotes, but the docs warn that an exploiter can trigger their events from any distance, at any time, often ignoring `Enabled` and `MaxActivationDistance`. A `ClickDetector` gets no server-side check at all, and of a prompt's events only `Triggered` gets a distance check. Treat their handlers like `OnServerEvent`: check the player on the server first.

**Key points**

- An exploiter controls their own client completely: any remote, any arguments, any rate, and their own character.
- Anything that replicates can be read, including LocalScripts and ModuleScripts that never run on the client.
- ServerScriptService and ServerStorage never reach a client, and a client's edits to the world stay on its own screen.
- `player` in `OnServerEvent` is always the real sender; every other argument is untrusted.
- `ClickDetector` and `ProximityPrompt` events need the same server-side checks as remotes.
