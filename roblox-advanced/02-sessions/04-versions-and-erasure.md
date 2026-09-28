# Versions, metadata and erasure

*Roll a key back to how it stood at a time, keep its user IDs and metadata on every write, and name keys so an erasure request finds them.*

Roblox Advanced · Sessions · 11 min

---

An update ships with a shop bug, and players' coins drop to 0 as they save. You fix it an hour later, and the support tickets arrive. The coins are not gone: a DataStore keeps a key's older versions. Getting one back safely takes the versioning APIs, plus the user IDs and metadata that every write has to carry.

## Rolling back a key

```luau
local key = "Player_" .. userId

-- Every kept version: Version, CreatedTime, IsDeleted
local ok, pages = pcall(store.ListVersionsAsync, store, key)

-- One version, by its id from that list
local ok2, value, info = pcall(store.GetVersionAsync, store, key, versionId)

-- Whatever was current at a moment, in Unix milliseconds
local beforeBug = DateTime.fromUniversalTime(2026, 9, 28, 13, 55).UnixTimestampMillis
local ok3, old, oldInfo = pcall(store.GetVersionAtTimeAsync, store, key, beforeBug)
```

Not every write is kept: the first write to a key in each UTC hour is kept as a backup, and older backups expire, while the latest version never does. `GetVersionAtTimeAsync` returns `nil` if no version was current then.

Every version carries the key's user IDs and metadata too. `SetAsync` takes them as arguments. An `UpdateAsync` transform returns them after the value, and if it returns the value alone, both are cleared, session lock included.

```luau
-- SetAsync: user IDs and custom metadata go in as arguments
local options = Instance.new("DataStoreSetOptions")
options:SetMetadata({ Region = "EU" })
pcall(store.SetAsync, store, key, value, { userId }, options)

-- UpdateAsync: the transform hands them back after the value
pcall(store.UpdateAsync, store, key, function(current, keyInfo)
	return newValue, keyInfo:GetUserIds(), keyInfo:GetMetadata()
end)
```

The docs say to set metadata on every write, even when it has not changed, or the current metadata is lost.

## Right to erasure

Players can ask Roblox to erase their data. Roblox then deletes the keys and stores that match your deletion templates, such as `Player_{UserId}`, with the requester's ID filled in. So the ID must appear in a key, store or scope name, in a fixed pattern; `userIds` alone does not qualify a key. The docs still recommend passing `userIds`, to help track a user's data.

> **Recognising older code**
>
> Older games save each session to a new key and track the newest in an OrderedDataStore: the "berezaa method", DataStore2's default. Versions do that job now, and every extra key counts against storage. Save to one key per player and roll back with `GetVersionAtTimeAsync`.

> **Or let a library do it**
>
> Scribe does this whole module for you: a session lock, a numbered migration chain, leaderboards, version restores and erasure. Kalo's Scribe course covers them from the library side in its modules 5 and 6. The mechanics here tell you what it is doing when a save fails.

**Key points**

- `ListVersionsAsync`, `GetVersionAsync` and `GetVersionAtTimeAsync` read a key's kept versions; roll back by writing the old value again.
- Restore only while nobody holds the player's live lock, or the owner's next save overwrites it.
- `SetAsync` takes user IDs and a `DataStoreSetOptions` for metadata; an `UpdateAsync` transform returns both after the value, or they are cleared.
- Erasure matches deletion templates against key, store and scope names, so put `User.Id` in the key name; pass `userIds` for tracking.
