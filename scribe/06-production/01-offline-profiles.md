# Offline profiles

*Read, edit, export and erase a stored profile for a player who is not here, and why every write fails closed.*

Scribe · Production Data · 10 min

---

A support ticket, a GDPR request, a backfill: all of them act on the stored profile rather than a live session. Everything here takes a `userId`, and every call yields, so it belongs in a command handler or an admin panel, never on a per-frame path. `GetOffline(userId)` is a read-only snapshot of the raw table with derived fields filled in; `UpdateOffline(userId, fn)` runs `fn` against a copy and commits it with a compare-and-set.

**An apology grant**

```luau
local snapshot = Data.GetOffline(ben.UserId)
print(snapshot.Coins, snapshot.Level) --> 100 1

local ok, reason = Data.UpdateOffline(ben.UserId, function(raw)
	raw.Gems = (raw.Gems or 0) + 100
end)
print(ok, reason)
```

| Call | Does | Notes |
| --- | --- | --- |
| `GetOffline(userId)` | a raw snapshot, derived fields filled in | reads a live session's clone when one is here |
| `UpdateOffline(userId, fn)` | edits a copy, commits by compare-and-set | fails closed while a session is live |
| `ListVersions(userId, limit?)` | the profile's saved versions, newest first | live DataStore only |
| `GetVersion(userId, id)` | one version's raw table | read before you restore |
| `RestoreVersion(userId, id, opts?)` | makes a version the current profile | keeps `_Scribe` unless `RollBackReserved` |
| `Export(userId)` | the profile as a JSON string | buffers base64-encoded |
| `Erase(userId)` | deletes the profile and its board entries | idempotent |

## The session lock

Every write (`UpdateOffline`, `RestoreVersion`, `Erase`) fails closed when the user has a live session on any server; a session whose last write is older than 630 seconds counts as dead. So the order for a ticket is: get the player out, act, let them rejoin. `UpdateOffline`'s commit lands only if the profile is still the one your callback was handed, which adds one reason, "profile changed while the update was being prepared": nothing was written, retry from a fresh read.

**Fails closed, then lands**

```luau
local function grant(raw)
	raw.Gems = (raw.Gems or 0) + 100
end

print(Data.UpdateOffline(101, grant)) --> false  user has an active session on this server

leave(ava)
task.wait(1)

print(Data.UpdateOffline(101, grant)) --> true
print(Data.GetOffline(101).Gems) --> 100
```

## Export, erase, versions

`Export(userId)` is the profile as a JSON string for handing to the requester. `Erase(userId)` deletes the profile and then the user's leaderboard entries, answers `(false, reason)` if a board key survived so you retry, and is idempotent. Version history needs a real DataStore: in Mock mode the list is empty, so the restore below runs against a live store only. A restore keeps the reserved `_Scribe` root (receipts, perks, cooldowns) from the live profile, because rolling back money is a mistake, not a fix; `RollBackReserved = true` is the opt-in.

**Export, then erase**

```luau
local json = Data.Export(101)
print(type(json), json:sub(1, 12)) --> string  {"Coins":100,

print(Data.Erase(101)) --> true
print(Data.GetOffline(101)) --> nil

-- Live only: the sandbox's mock store keeps no history.
local versions = Data.ListVersions(101, 10)
local raw = Data.GetVersion(101, versions[1].Id)
if raw.Inventory ~= nil then
	Data.RestoreVersion(101, versions[1].Id)
end
```

**Key points**

- Every offline call takes a `userId` and yields; keep them off per-frame paths.
- `UpdateOffline`, `RestoreVersion` and `Erase` fail closed while a session is live anywhere.
- `UpdateOffline` commits by compare-and-set: a changed profile means nothing was written.
- `Export` is the GDPR answer, `Erase` the deletion, and both are safe to repeat.
- Version history is a live-store feature; a restore keeps `_Scribe` unless you opt out.
