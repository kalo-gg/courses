# DataStores

*A value under a key that outlives every server, read and written by web requests that can fail.*

Roblox Advanced · DataStores · 10 min

---

Ada earns 250 coins, leaves, and rejoins to find 0. Attributes and the tables you key by player all go when the server does.

A **DataStore** keeps a value under a key, outside every server, so the next server can read it back. Each read and write is a web request: it takes time, and it can fail.

## A store, a key, a value

```luau
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local store = DataStoreService:GetDataStore("PlayerData")

Players.PlayerAdded:Connect(function(player)
	local key = "Player_" .. player.User.Id
	local ok, data = pcall(store.GetAsync, store, key)
	print(ok, data) --> true nil   (nothing saved yet: a new player)
end)

Players.PlayerRemoving:Connect(function(player)
	local key = "Player_" .. player.User.Id
	local ok, err = pcall(store.SetAsync, store, key, { Coins = 250 })
end)
```

`GetDataStore` returns the same store for the same name on every server. `pcall(store.GetAsync, store, key)` is the call `store:GetAsync(key)`, but an error comes back as `false` and a message instead of ending the thread.

So a load has three outcomes, not two. Only a key that was never written gets defaults. A failed request says nothing about the data, so the player stays unloaded and nothing is saved for them.

| `pcall` returns | What it means | What to do |
| --- | --- | --- |
| `false, message` | The request failed. The data is unknown. | Mark the player as not loaded. Never save over their key. |
| `true, nil` | The key has never been written. | A new player: start from defaults. |
| `true, value` | The saved data. | Use it. |

## The cache, and Studio

`GetAsync` keeps what it read in a local cache for 4 seconds, and reading the same key again in that time returns the copy without asking Roblox. A cached read costs nothing against the limits in lesson 3, but it can be stale if another server wrote the key since. To read what the backend holds, pass a `DataStoreGetOptions` with `UseCache = false`.

```luau
local fresh = Instance.new("DataStoreGetOptions")
fresh.UseCache = false

local ok, data = pcall(store.GetAsync, store, key, fresh) -- always asks Roblox
```

An uncached read always counts against the request budgets, so keep it for when you need it, such as checking a write that errored (lesson 3).

Studio cannot reach DataStores until the place is published and **Enable Studio Access to API Services** is on, in Experience Settings ⟩ Security. Studio then reads and writes the *same* data as the live game, so the docs advise turning it on only in a separate test experience. This course's exercises run against a fake cloud that fails on demand.

> **Recognising older code**
>
> `store:GetAsync(player.UserId)` with no `pcall` is older code. `UserId` is not deprecated, and a player who joined before domain-scoped IDs keeps the same id, so keys built from it still work. New code builds keys from `player.User.Id`, which the docs name the standard identifier, and wraps every call in `pcall`.

**Key points**

- `DataStoreService:GetDataStore(name)` returns the same store on every server, and each key holds one value.
- Build a player's key from `player.User.Id`, and wrap every call in `pcall`.
- A failed load is not a new player: keep the player unloaded and never save over their key.
- `GetAsync` answers from a 4-second cache; `UseCache = false` asks the backend.
