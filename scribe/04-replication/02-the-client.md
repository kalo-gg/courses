# The client half

*The mirror reads defaults until the snapshot lands, Observe is the whole of a reactive UI, and a client write never leaves the client.*

Scribe · Replication & Commands · 11 min

---

The client reads the same accessor tree with the same API, on the client object itself: `Data.Coins.Observe(fn)`. It never errors and never makes you wait: reads return template defaults until the first snapshot lands, then `Observe` and `Changed` fire with the real values. For one-shot startup logic, `Data.IsReady()` does not yield and `Data.WaitForData(timeout?)` yields up to 30 seconds and returns a boolean. Seven methods do yield until the snapshot, `Request` and `Owns` among them.

**StarterPlayerScripts/Hud.client.luau**

```luau
local Data = require(ReplicatedStorage.Shared.EmberfallData).Client

Data.Coins.Observe(function(coins)
	coinsLabel.Text = tostring(coins)
end)

Data.Level.Observe(function(level)
	levelLabel.Text = "Level " .. level
end)

if Data.WaitForData() then -- true once the snapshot has landed, false after 30 s
	showMainMenu(Data.Level.Get())
end
```

## In the sandbox: asClient

There is one process here, so `asClient(player, fn)` runs `fn` as that player's client: `RunService:IsServer()` is false inside it and `Players.LocalPlayer` is the player, so `asClient(ava, function() return Scribe(options).Client end)` builds the real client half from the same options. The Default transport's RemoteEvents loop back, so the handshake lands before `asClient` returns, and every diff after it arrives on the next frame: a `task.wait()` sits between a server write and the client read.

**Server and client, one script**

```luau
local Client = asClient(ava, function()
	return Scribe(options).Client
end)
print(Client.WaitForData()) --> true

Client.Coins.Observe(function(coins)
	print("client sees", coins)
end) --> client sees 100

data.Coins.Increment(40) -- on the server
task.wait() -- one frame: one replication flush
--> client sees 140
```

## Other players' shared data

`Shared` roots stream to everyone. `Data.GetShared(playerOrUserId)` hands you every Shared root keyed by its template name, so you read `shared.Nameplate.Title`; `Data.OnSharedChanged` fires `(userId, shared)` on every change, and with nil once they leave, a user id because the owner may already be gone. The server broadcasts a player's Shared roots to every client except that player's own, so handle the local player on the accessor and everyone else on `GetShared`.

**Ben's title, seen by Ava**

```luau
local ben = join("Ben", 102)
Data.WaitForData(ben).Nameplate.Title.Set("Ember Knight")
task.wait(1)

print(Client.GetShared(ben).Nameplate.Title) --> Ember Knight

Client.OnSharedChanged:Connect(function(userId, shared)
	print(userId, shared and shared.Nameplate.Title)
end)
Data[ben].Nameplate.Title.Set("Ember Lord")
task.wait(1) --> 102  Ember Lord
leave(ben)
task.wait(1) --> 102  nil
```

**Key points**

- The client reads the same tree on the client object; reads never yield and never error.
- `Observe` is all a reactive UI needs; `IsReady` and `WaitForData` (a boolean) gate one-shot logic.
- A client write is optimistic and local; the next server diff overwrites it.
- `GetShared` and `OnSharedChanged` are how you read everyone else's Shared roots; your own come through the accessor.
- In the sandbox, `asClient` builds the client and one frame delivers a diff.
