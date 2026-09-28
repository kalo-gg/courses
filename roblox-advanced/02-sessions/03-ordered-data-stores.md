# Leaderboards with OrderedDataStore

*Integer scores in an OrderedDataStore, read back sorted a page at a time, and how often a server can afford to write and read them.*

Roblox Advanced · Sessions · 10 min

---

A global leaderboard needs the top ten out of everyone who ever played. A DataStore cannot answer that: reading each player's key would cost a request per player and still leave you to sort them.

An **OrderedDataStore** keeps one integer per key and hands them back sorted, a page at a time.

## Writing scores

```luau
local DataStoreService = game:GetService("DataStoreService")
local wins = DataStoreService:GetOrderedDataStore("Wins")

-- Called from the player's save: one write per save, not one per win.
local function saveWins(player: Player, count: number)
	local ok, err = pcall(wins.SetAsync, wins, "Player_" .. player.User.Id, count)
	if not ok then
		warn("Wins not saved:", err)
	end
end
```

Values must be integers. Ordered stores keep no versions, metadata or user IDs, so their `DataStoreKeyInfo` is always `nil`. A key built from `User.Id` leads each rank back to a player.

## Reading the top

```luau
local ok, pages = pcall(wins.GetSortedAsync, wins, false, 10) -- false: highest first
if ok then
	for rank, entry in pages:GetCurrentPage() do
		print(rank, entry.key, entry.value) --> 1  Player_2918475  86
	end
end
```

`GetCurrentPage` is an array of `{ key, value }` tables. `AdvanceToNextPageAsync` fetches the next page, until `IsFinished` is true.

Ordered stores have their own server budgets, below. `GetSortedAsync` spends the small `OrderedList` one, and every player's board shows the same top ten, so read it on one timer for the whole server, about once a minute. Write a score when the player's data saves, not on every point.

| Request type | Spent by | Per server |
| --- | --- | --- |
| `OrderedWrite` | `SetAsync`, `IncrementAsync`, the write of `UpdateAsync` | (30 + 5 × players in this server) requests per minute |
| `OrderedList` | `GetSortedAsync` | (5 + 2 × players in this server) requests per minute |
| `OrderedRead` | `GetAsync`, the read of `UpdateAsync` | (60 + 40 × players in this server) requests per minute |

For comparison, a standard store's `StandardWrite` allows (60 + 40 × players in this server) requests per minute. `GetRequestBudgetForRequestType(Enum.DataStoreRequestType.OrderedList)` shows what is left.

`GetSortedAsync` can be throttled or fail like any DataStore call. Keep the last good board and try again next time: swapping in an empty list on an error blanks the leaderboard for everyone.

**Key points**

- An OrderedDataStore holds one integer per key; scale decimals such as times to whole numbers.
- `GetSortedAsync(ascending, pageSize)` returns pages of `{ key, value }`, sorted by value.
- Ordered stores have their own, smaller budgets, and no versions, metadata or user IDs.
- Write a score when the player saves; read the board on one timer per server; keep the last good board when a read fails.
