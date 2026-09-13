# What is Scribe?

*Write the shape of your data once and Scribe saves it, locks it, replicates it and types it.*

Scribe · Scribe Foundations · 8 min

---

A coin balance has to survive a rejoin, so you write `GetAsync` and `SetAsync` behind `pcall`s. Two servers can hold one player, so you write a lock. The HUD shows the balance, so you write a RemoteEvent. A purchase must never be lost, so you write a save on leave and an autosave.

By the time the shop works, you have written a small database. **Scribe** is that database, already written: describe the data once, in a plain table, and it does the rest.

## By hand, then with Scribe

**By hand: the pieces you end up writing**

```luau
local store = DataStoreService:GetDataStore("PlayerData")

Players.PlayerAdded:Connect(function(player)
	local ok, saved = pcall(store.GetAsync, store, "PLAYER_" .. player.UserId)
	local data = (ok and saved) or { Coins = 0 }
	-- a lock, so another server cannot load the same player
	-- an autosave loop, so a crash costs minutes and not the session
	coinsRemote:FireClient(player, data.Coins) -- and again on every change
	sessions[player] = data
end)

Players.PlayerRemoving:Connect(function(player)
	pcall(store.SetAsync, store, "PLAYER_" .. player.UserId, sessions[player])
end)
```

**With Scribe: one shared module and a wait**

```luau
-- ReplicatedStorage/Shared/EmberfallData.luau
local Scribe = require(game:GetService("ReplicatedStorage").Packages.Scribe)

return Scribe({
	Template = {
		Coins = Scribe.Int(0, { Min = 0 }),
		Xp = Scribe.Int(0, { Min = 0 }),
		Level = Scribe.Derived(Scribe.Int(1), { "Xp" }, function(xp)
			return math.floor(xp / 1000) + 1
		end),
	},
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
})

-- ServerScriptService/Emberfall.server.luau
local Data = require(ReplicatedStorage.Shared.EmberfallData).Server

Players.PlayerAdded:Connect(function(player)
	local data = Data.WaitForData(player)
	if data then
		data.Coins.Increment(50)
	end
end)
```

`Increment` validates against the declared minimum, sends the change to that player's client and marks the profile for the next save. Nothing else is written.

Scribe has four jobs. **Persistence** sits on ProfileStore's session lock: one server holds a profile at a time, and every save writes the whole tree. **Validation** comes from the declarators. **Replication** streams schema-compressed diffs to the owning client over two RemoteEvents Scribe makes itself. **Typing** is the accessor tree, checked by the new Luau solver.

A patched copy of ProfileStore 1.0.3 is vendored inside the package: nothing else to install, and a ProfileStore your game already uses is left alone.

| Job | Scribe API | Where in the course |
| --- | --- | --- |
| Persist a profile | `Scribe({ Template = … })`, `Data.WaitForData` | Your first session |
| Validate a write | `Scribe.Int(0, { Min = 0 })` | Declarators |
| Read on the client | `Data.Coins.Observe` | The client half |
| Change from the client | `Data.Command`, `Data.Request` | Commands |
| Compute a field | `Scribe.Derived` | Derived fields |

## What it is not

Not a UI framework (it ships adapters for Fusion, React and Vide), not an ECS, not an anti-cheat, not a ban list. It is the data layer, and the boundary between what a client claims and what the server decides.

One option, `Mode`, swaps the whole stack onto an in-memory store. Every lesson here runs that way, in your browser, against the real 2.3.0 sources.

> **The docs' example game is this course's example game**
>
> Scribe's guides are built around Emberfall, a small adventure RPG. Ava is user 101 and Ben is 102. The template you meet in two lessons is the one the docs use on every page, so what you learn here reads straight across.

**Key points**

- One table declares the data; Scribe saves it, locks it, replicates it and types it.
- ProfileStore is inside the package, patched, and needs no separate install.
- Every read and write goes through an accessor method, never a plain table.
- The client only ever reads; changing data is a command the server answers.
