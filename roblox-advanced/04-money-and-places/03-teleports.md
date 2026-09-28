# Teleports

*Send a party to one reserved server with TeleportAsync, and catch both the call that throws and the teleport that fails later.*

Roblox Advanced · Money and places · 11 min

---

The lobby fills a party of three and sends them to the arena. One call throws because the service hiccuped, and nobody moves. Another starts, then fails for one player on the loading screen, who is left alone in the lobby while the others play without them.

Teleports cross servers, so they fail like any web call, and in one way of their own. They also do not run in Studio playtests: test them in a published experience.

## TeleportAsync and TeleportOptions

```luau
local TeleportService = game:GetService("TeleportService")

local ARENA_PLACE = 1818 -- a place in this experience

local function sendToArena(party: { Player })
	local options = Instance.new("TeleportOptions")
	options.ShouldReserveServer = true -- a new server only this call can reach
	return pcall(TeleportService.TeleportAsync, TeleportService, ARENA_PLACE, party, options)
end
```

`TeleportAsync` runs only on the server and moves up to 50 players per call, all to the same destination. With options passed, it returns a `TeleportAsyncResult` that names where they went.

| Set on TeleportOptions | The players go to |
| --- | --- |
| Nothing | A public server, matched on the first player in the list |
| `ShouldReserveServer = true` | A new reserved server |
| `ReservedServerAccessCode = code` | The reserved server that code opens |
| `ServerInstanceId = jobId` | That public server |

`ShouldReserveServer` cannot be combined with the other two. `TeleportService:ReserveServerAsync(placeId)` returns an access code without moving anyone, and the code stays valid indefinitely.

A reserved server is one only teleports with its code can reach. Inside it, `game.PrivateServerId` is not empty and `game.PrivateServerOwnerId` is 0, which is how the arena's scripts can tell where they are running.

## Two ways to fail

The call itself can throw, for bad arguments or a network error, so it goes in `pcall`, and the teleport guide retries after a short wait. Or the call succeeds and the teleport fails later: the player stays in this server, and `TeleportService.TeleportInitFailed` fires once for each player it failed for, with the result, a message and a new `TeleportOptions` aimed at the same destination.

```luau
local RETRY_DELAY = 1

local function teleport(players: { Player }, options: TeleportOptions): boolean
	for attempt = 1, 2 do
		local ok, err = pcall(TeleportService.TeleportAsync, TeleportService, ARENA_PLACE, players, options)
		if ok then
			return true
		end
		warn("Teleport failed:", err)
		task.wait(RETRY_DELAY)
	end
	return false
end

TeleportService.TeleportInitFailed:Connect(function(player, result, message, placeId, options)
	if result ~= Enum.TeleportResult.Failure and result ~= Enum.TeleportResult.Flooded then
		warn(player.Name, "cannot go:", result.Name, message) -- the teleport itself is invalid
		return
	end
	task.wait(if result == Enum.TeleportResult.Flooded then 15 else RETRY_DELAY)
	teleport({ player }, options) -- only this player; the others have gone
end)
```

Like the teleport guide, this waits longer after `Flooded` and reports, rather than retries, results that mean the teleport itself is invalid. Cap the retries too: a player whose teleport keeps failing should stay and be told.

> **Recognising older code**
>
> `TeleportToPrivateServer`, `TeleportToPlaceInstance`, `TeleportPartyAsync` and `ReserveServer` are deprecated: `TeleportAsync` with `TeleportOptions`, and `ReserveServerAsync`, replace them. `TeleportService:Teleport(placeId)` called from a LocalScript is deprecated too: fire a remote and teleport from the server.

**Key points**

- `TeleportAsync(placeId, players, options)` is the one teleport method: server only, up to 50 players per call, one destination.
- Reserve with `ReserveServerAsync` when you may send people to the same server again; the code stays valid.
- A call can throw (use `pcall` and retry), or start and fail later (`TeleportInitFailed`, once per player left behind).
- Teleports do not run in Studio playtests; test them in a published experience.
