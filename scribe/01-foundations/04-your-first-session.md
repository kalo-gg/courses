# Your first session

*A player joins, the profile loads, a write lands, they leave, and the number is still there when they come back.*

Scribe · Scribe Foundations · 11 min

---

Profiles load asynchronously: Scribe fetches the stored data, reconciles it against the template and takes a session lock, and all of that takes time. `Data.WaitForData(player, timeout?)` yields until the profile is Ready and returns `(accessor?, reason?)`.

Handle the nil branch. The most common reason is `player-left`, which is routine and needs no handling beyond returning.

**ServerScriptService/Emberfall.server.luau**

```luau
local Data = require(ReplicatedStorage.Shared.EmberfallData).Server

Players.PlayerAdded:Connect(function(player)
	local data = Data.WaitForData(player)
	if not data then
		return -- the load failed or the player left; Scribe is already handling it
	end

	data.Coins.Increment(50)
	data.Xp.Increment(1200)
	print(data.Level.Get()) --> 2, recomputed from Xp for you
end)
```

## In the sandbox

There is no Play button here. `join("Ava", 101)` creates a Player, parents it to `Players` and fires `PlayerAdded`; `leave(ava)` fires `PlayerRemoving`; `task.wait(seconds)` moves a virtual clock. The store is ProfileStore's own in-memory mock, so a profile persists across leave and rejoin within one run.

One warning to expect: ten seconds into a script with no client, Scribe warns `CLIENT_HANDSHAKE_TIMEOUT`, the same warning a real place shows when no LocalScript required the bundle. You build the client in the Replication module; until then, read past it.

**A session, end to end**

```luau
Players.PlayerAdded:Connect(function(player)
	local data = Data.WaitForData(player)
	if not data then
		return
	end
	data.Coins.Increment(50)
end)

local ava = join("Ava", 101)
task.wait(0.1) -- the handler runs on its own thread, a moment after the load lands
print(Data[ava].Coins.Get()) --> 50

leave(ava)
task.wait(1)

local again = join("Ava", 101)
task.wait(0.1)
print(Data[again].Coins.Get()) --> 100
```

The handler ran on both joins, and the second load read what the first session saved. Same code in Mock mode and against a real DataStore; the short waits only give the handler its turn.

Between `leave` and the rejoin, the session ended, Scribe ran the final save, released the lock, and the next load read the stored profile. In Mock mode the store is a table in memory; in Live mode it is the DataStore. Your code does not change.

## Reads that never yield

**Once it is Ready**

```luau
print(Data.GetState(player)) --> "Loading", "Ready" or "SessionEnded"; never yields

local data = Data.Get(player) -- the accessor; errors while Loading
local same = Data[player] -- the same accessor, as sugar
```

> **Always handle the nil branch**
>
> `if not data then return end` is the whole handling most games need. Of the seven reasons a wait can fail, only `Scribe.Reason.StillLoading` is worth a retry; the Sessions module has the table.

**What you can do now**

- Install Scribe and return the bundle from one shared module.
- Read a template and tell a plain value from a declarator.
- Wait for a profile with `WaitForData` and handle the nil branch.
- Read a loaded profile with `Data[player]` and its state with `GetState`.
- Drive a session in the sandbox with `join`, `leave` and `task.wait`.
