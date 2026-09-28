# Joining with data

*Send only a server-made id, keep the party in MemoryStore, release the lock before leaving, and check the claim on arrival.*

Roblox Advanced · Money and places · 11 min

---

The lobby sends a party to the arena with everything attached: the mode, each player's loadout, maybe their coins. The arena reads it and all is well, until someone edits it on the way. Teleport data travels through the player's client, so a modified client can change what arrives.

So send an id, and keep what matters on the servers.

## What GetJoinData guarantees

```luau
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
	local joinData = player:GetJoinData()
	print(joinData.SourcePlaceId) -- the place that sent them
	print(joinData.TeleportData) -- what the sending server set, carried by the client
end)
```

`GetJoinData` is read on the server. A player who joined without a teleport has no `TeleportData`, so check before you index it.

The docs promise three things about this teleport data: a Roblox server sent it in the last 48 hours, it was sent with this player, and `SourcePlaceId` is the real place it came from. So you can tell it came from your lobby.

They do not promise it arrived unchanged: the same page warns that the client transmits it, so an exploiter can still abuse it. So the lobby stores the real party in MemoryStore under a new id and sends only that id. The arena looks it up: a forged id finds nothing, and a real one names a party the player must be in.

## The lobby's half

```luau
local HttpService = game:GetService("HttpService")
local parties = game:GetService("MemoryStoreService"):GetHashMap("Parties")

local function sendParty(party: { Player }, mode: string, accessCode: string): boolean
	local partyId = HttpService:GenerateGUID(false)
	local members = {}
	for _, player in party do
		table.insert(members, player.User.Id)
	end
	if not pcall(parties.SetAsync, parties, partyId, { members = members, mode = mode }, 600) then
		return false -- nobody has moved yet
	end

	for _, player in party do
		Profiles.release(player) -- save, and clear this server's lock
	end
	local options = Instance.new("TeleportOptions")
	options.ReservedServerAccessCode = accessCode
	options:SetTeleportData({ partyId = partyId })
	return teleport(party, options) -- the retrying helper from the last lesson
end
```

The id is all the client carries. The record expires by itself after 10 minutes, so an abandoned party cleans itself up.

Release the lock before the teleport, not after. The docs name teleports as the classic quick hop from one server to another: if the lobby still holds the lock when the arena loads, the arena must wait for it or ask for it. If the teleport then fails for good, claim the lock again before the player plays on.

> **Recognising older code**
>
> Older tutorials send a player's stats as teleport data and load them straight in: `profile.Coins = player:GetJoinData().TeleportData.coins`. Anyone who edits their client edits their coins. Send an id, and keep the data in a DataStore or MemoryStore.

**Key points**

- `GetJoinData()` guarantees who sent the teleport data and from where, not that the client left it unchanged.
- Send only a server-made id; keep the real data in MemoryStore or a DataStore and look it up on arrival.
- Admit a player only if the record exists and lists their `User.Id`; reject everything else.
- Save and release the session lock before the teleport, and claim it again if the teleport fails for good.
