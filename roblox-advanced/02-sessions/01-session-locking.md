# Session locking

*One server owns a player's data at a time: a lock in the key's metadata, claimed with UpdateAsync, refreshed by every save, and taken over when it goes stale.*

Roblox Advanced · Sessions · 11 min

---

Ada gives Bo her sword and leaves. Her server's final save is throttled and waits in a queue, and before it lands she joins another server, which loads her data with the sword still in it. The queued save lands, then the new server's autosave writes over it. Bo has the sword, and so does Ada.

Every call did what it was told. The bug is that two servers held Ada's data at once. A **session lock** gives her data one owner.

## A lock in the key's metadata

**Claim the key as you load it**

```luau
local HttpService = game:GetService("HttpService")
local DataStoreService = game:GetService("DataStoreService")
local store = DataStoreService:GetDataStore("PlayerData")

local LOCK_EXPIRY = 30 * 60 -- seconds; your choice, well above the autosave interval

-- The saved data and this session's lock id, or nil if another server owns the key.
local function claim(userId: number)
	local lockId = HttpService:GenerateGUID(false)
	local claimed = false
	local ok, data = pcall(store.UpdateAsync, store, "Player_" .. userId, function(saved, keyInfo)
		claimed = false -- the transform can run twice; only the last run counts
		local metadata = if keyInfo then keyInfo:GetMetadata() else {}
		if metadata.LockId and metadata.JobId ~= game.JobId
			and os.time() - metadata.LockTime < LOCK_EXPIRY then
			return nil -- another server's live lock: write nothing
		end
		metadata.LockId, metadata.JobId, metadata.LockTime = lockId, game.JobId, os.time()
		claimed = true
		return saved or {}, if keyInfo then keyInfo:GetUserIds() else { userId }, metadata
	end)
	if ok and claimed then
		return data, lockId
	end
	return nil
end
```

`GenerateGUID` gives this session a lock id no other session shares. The check and the claim are one `UpdateAsync`, so no server can claim in between. `JobId` names the owner, so another server can later ask it to let go. A new player gets `{}`, not `nil`: returning `nil` would cancel the write, and the lock with it.

A server that crashes never releases its locks, so a lock must expire. Every save rewrites `LockTime`, and any server may take over a lock older than the expiry. The expiry is yours to choose. Keep it well above the autosave interval, or a live owner looks dead between two saves.

| The key's metadata holds | `claim` does | Because |
| --- | --- | --- |
| No `LockId` | Claims it | Nobody owns the data, or its owner released it. |
| Another server's lock, `LockTime` recent | Returns `nil`, writes nothing | That server is playing this player right now. |
| Another server's lock, older than `LOCK_EXPIRY` | Takes it over | Its owner stopped saving, most likely a crash. |
| This server's own lock | Claims it again | A quick rejoin to the same server. |

## Saving as the owner

Owning the lock is half the job. Every later save goes through `UpdateAsync` too, and writes only if the metadata still holds this session's `LockId`. If it does not, another server took over, and this server's copy is out of date.

**Save, refresh and release**

```luau
local function save(userId: number, data, lockId: string, release: boolean?): boolean
	local saved = false
	pcall(store.UpdateAsync, store, "Player_" .. userId, function(_, keyInfo)
		saved = false
		local metadata = if keyInfo then keyInfo:GetMetadata() else {}
		if metadata.LockId ~= lockId then
			return nil -- the lock was taken over: this copy is out of date
		end
		if release then
			metadata.LockId, metadata.JobId, metadata.LockTime = nil, nil, nil
		else
			metadata.LockTime = os.time() -- keep the lock fresh
		end
		saved = true
		return data, keyInfo:GetUserIds(), metadata
	end)
	return saved
end
```

The autosave calls `save(…, false)`, which keeps the lock fresh. `PlayerRemoving` and `BindToClose` pass `true`. A `false` result means the lock is gone: stop saving that player.

> **Check and claim in one call**
>
> A `GetAsync` to look at the lock, then a `SetAsync` to take it, leaves a gap: two servers can both see no lock and both claim. `UpdateAsync` reads and writes the key as one step, and runs the transform again if another write landed in between.

**Key points**

- Two servers can hold one player for a moment (a fast rejoin, a teleport, a queued save), and the later save overwrites the newer data.
- The lock lives in the key's metadata (`LockId`, `JobId`, `LockTime`) and is checked and written in the same `UpdateAsync` that loads the data.
- Every save checks that the `LockId` is still this session's, refreshes `LockTime`, and the last save clears the lock.
- A lock older than your expiry is stale and may be taken over; keep the expiry well above the autosave interval.
- Reset any flag the transform sets at the top of the transform: `UpdateAsync` can run it twice.
