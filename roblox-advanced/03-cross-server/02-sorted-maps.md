# MemoryStore sorted maps

*Fast shared data that every server can read in order, and that expires on its own when nobody refreshes it.*

Roblox Advanced · Across servers · 11 min

---

The lobby needs a server browser: every open server and how many players it has, kept up to date. A DataStore is the wrong tool. Its reads are cached for 4 seconds, every write spends a budget, and a server that crashes leaves its row behind for good. A message is no better: a server that starts later has missed it.

**MemoryStoreService** holds fast data that every server shares, and every item in it expires. Its **sorted map** keeps its keys in order.

## Set, expire, read in order

```luau
local MemoryStoreService = game:GetService("MemoryStoreService")
local Players = game:GetService("Players")

local servers = MemoryStoreService:GetSortedMap("OpenServers")

local count = #Players:GetPlayers()
pcall(servers.SetAsync, servers, game.JobId, { players = count }, 30, count)

-- the ten fullest servers
local ok, items = pcall(servers.GetRangeAsync, servers, Enum.SortDirection.Descending, 10)
-- items[1] = { key = "<a JobId>", value = { players = 38 }, sortKey = 38 }
```

`SetAsync(key, value, expiration, sortKey)`: the item disappears `expiration` seconds later, and the optional sort key orders it. `GetRangeAsync(direction, count)` returns up to `count` items in order.

Items sort by sort key first: numbers, then strings, then items with no sort key. Items that tie sort by key. The optional bounds of `GetRangeAsync` are exclusive, so passing the last item you read as a bound gets the next page.

An expiration can be at most 3,888,000 seconds (45 days), but a server list wants seconds. Nothing in a memory store is backed up, so keep anything that must last in a DataStore.

## UpdateAsync

**Kills this hour, counted from every server**

```luau
local kills = MemoryStoreService:GetSortedMap("KillsThisHour")

local function addKill(player: Player)
	local ok, err = pcall(kills.UpdateAsync, kills, tostring(player.User.Id), function(old, _sortKey)
		local n = (old or 0) + 1
		return n, n -- the new value, and the new sort key
	end, 3600)
	if not ok then
		warn("Kill not counted:", err)
	end
end
```

The transform gets the current value and sort key, or `nil` for a new key. If another server changes the item before the write lands, the transform runs again with the newer value, as a DataStore's does. Return `nil` to cancel.

Every call is a network request that can fail or be throttled. In a heartbeat loop, a failed write should cost one beat, not the loop: catch the error, and the next beat writes again. When the server closes, remove its entry; expiration covers the crash.

**Key points**

- `MemoryStoreService:GetSortedMap(name)` gives every server the same map.
- `SetAsync(key, value, expiration, sortKey)`: every item expires, after at most 45 days.
- Items sort by sort key (numbers, then strings, then none), then by key. `GetRangeAsync` reads them in order.
- `UpdateAsync` re-runs its transform when another server changed the item first.
- A short expiration plus a regular rewrite is a heartbeat: a dead server's entry ages out by itself.
