# UpdateAsync

*Read, change and write a key in one call, which runs again when another server wrote first.*

Roblox Advanced · DataStores · 11 min

---

Ada's friend gifts her 50 coins from another server. That server reads her key (100 coins), adds 50 and writes 150. A moment later Ada's own server, which read 100 when she joined, saves 120 after a quest. The last write wins, and the gift is gone.

`SetAsync` writes blind: it never looks at what the key holds now. When more than one server can write a key, change it with `UpdateAsync`.

## Read, change, write

```luau
local function addCoins(player: Player, amount: number): boolean
	local key = "Player_" .. player.User.Id
	local ok, err = pcall(store.UpdateAsync, store, key, function(data)
		data = data or { Coins = 0 }
		data.Coins += amount
		return data
	end)
	if not ok then
		warn("Could not add coins: " .. tostring(err))
	end
	return ok
end
```

The **transform** receives the key's current value (`nil` for a new key) and returns the value to write. It starts from what is stored now, not from a copy this server read earlier.

Back to the gift. Both servers now add to what is stored. If the gift lands first, Ada's server runs its transform again on 150 and writes 170. Nothing is lost, and the servers never had to talk to each other.

The cost: one call spends a read *and* a write from the server's budgets, which lesson 3 covers.

## Rules for a transform

Because a transform can run more than once, it follows three rules:

- **It cannot yield.** No `task.wait`, no other DataStore call, nothing that waits.
- **It has no side effects.** Do not grant items or fire remotes inside it. Set a local, and act after the call returns.
- **It returns `nil` to cancel.** Nothing is written.

It also receives a second argument, a `DataStoreKeyInfo` with the key's version, user IDs and metadata. Module 2 builds session locks on it.

```luau
local bought = false
local ok = pcall(store.UpdateAsync, store, key, function(data)
	bought = false -- a later run replaces an earlier run's answer
	if data == nil or data.Coins < PRICE then
		return nil -- cancel: nothing is written
	end
	data.Coins -= PRICE
	bought = true
	return data
end)

if ok and bought then
	giveSword(player) -- once, after the write landed
end
```

`bought` is reset at the top of the transform, so only the run whose result was written decides.

|  | `SetAsync` | `UpdateAsync` |
| --- | --- | --- |
| Reads the key first | No | Yes, and again if another server writes meanwhile |
| Spends | A write | A read and a write |
| Can cancel | No | Return `nil` |
| Use it for | A quick write of a key no other server writes at the same time | Any key more than one server may write |

**Key points**

- `SetAsync` writes without reading, so it can overwrite a newer value another server saved.
- `UpdateAsync` passes the current value to your transform and writes what it returns; if another server wrote in between, it runs the transform again.
- A transform cannot yield and must have no side effects; returning `nil` cancels the write.
- One `UpdateAsync` call spends both a read and a write.
