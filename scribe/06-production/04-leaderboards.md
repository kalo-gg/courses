# Leaderboards

*Name a field, and Scribe keeps every player's score in an OrderedDataStore and hands you the top hundred from a cache.*

Scribe · Production Data · 10 min

---

A leaderboard is an all-time global ranking of one numeric field. `Leaderboards = { TopLevel = { Stat = "Level", Limit = 100, Replicate = true } }` is all of it: Scribe watches `Level` on every loaded player, queues a write when it changes, and re-reads the top `Limit` in the background. `Level` is derived and still rankable, because its input persists. `GetLeaderboard(name, n)` returns `{ Rank, UserId, Name, Score }` entries; `GetMyRank(player, name)` is a position within the top `Limit`, or `nil` outside it or before the first refresh.

**Declare, read, react**

```luau
Leaderboards = {
	TopLevel = { Stat = "Level", Limit = 100, Replicate = true },
},

-- later, on the server
for _, entry in Data.GetLeaderboard("TopLevel", 10) do
	print(entry.Rank, entry.Name, entry.Score)
end
print(Data.GetMyRank(player, "TopLevel")) --> 7, or nil

Data.OnLeaderboard:Connect(function(boardName, entries)
	print(boardName, "refreshed with", #entries, "entries")
end)
```

## When it fills

Every read hits a cache, never the store, so a board is empty until its first refresh lands: the loop starts about five seconds after boot and staggers boards across their interval. `RefreshInterval` is 60 seconds by default and can only go slower; the ceiling is 12 reads a minute summed over every board. In the sandbox a short wait after the write lets the first refresh land; in Mock mode the store is in memory and discarded with the session.

**One entry, after the first refresh**

```luau
data.Coins.Set(90)
print(#Data.GetLeaderboard("TopCoins")) --> 0   (nothing has refreshed yet)

task.wait(8)
for _, entry in Data.GetLeaderboard("TopCoins") do
	print(entry.Rank, entry.Name, entry.Score) --> 1  Ava  90
end
```

| Key | Default | Notes |
| --- | --- | --- |
| `Stat` | required | a dotted path to an `Int` or a `Big` |
| `Limit` | 100 | entries kept, 1 to 100 |
| `Scale` | 1 | stores `round(value * Scale)`; refused on a Big |
| `SigFigs` | Big only | resolution against range for a Big board |
| `Replicate` | false | mirror the top to clients; refused on a Big |
| `RefreshInterval` | 60 | seconds between reads; can only go slower |
| `StoreName` | `LB_<name>` | the OrderedDataStore behind the board |

## What a Stat may be

A dotted path to a number field, an `Int` or a `Big`, checked at startup: not a string, boolean, enum, flags or container, not a `Session` field, not a derived field reading session state. `Scale` stores `round(value * Scale)` for fractional stats and is refused on a Big. A Big board ranks exactly, with `SigFigs` deciding resolution against range, and is server-only. A typo in a dynamic container path cannot be caught and leaves a board permanently empty.

**A Big board, and a fractional one**

```luau
Leaderboards = {
	TopEssence = { Stat = "Essence", SigFigs = 14 },
	TopPlaytime = { Stat = "Stats.Playtime", Scale = 100 },
},

for _, entry in Data.GetLeaderboard("TopEssence", 10) do
	local score = entry.Score
	if type(score) == "table" then
		print(entry.Rank, entry.Name, score:Short()) --> 1  Ava  1.2M
	end
end
```

**Key points**

- One config line per board: a `Stat` path, a `Limit`, and whether clients see it.
- Reads come from a cache the refresh loop fills; a board is empty until its first refresh.
- `GetMyRank` is a rank within the cached top, or nil; there is no exact global rank.
- `RefreshInterval` only goes slower, and the fleet-wide ceiling is 12 reads a minute.
- A Stat is an `Int` or a `Big` on a persisted path; `Scale` handles fractions, `SigFigs` a Big.
