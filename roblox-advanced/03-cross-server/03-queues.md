# MemoryStore queues

*One shared line: each read hides its items from other servers, and an item nobody removes comes back by itself.*

Roblox Advanced · Across servers · 11 min

---

Four lobby servers each have a few players waiting for a 4-player round. If each server matches only its own players, a server with three waits forever while another has one to spare. The lobby needs one shared line.

A sorted map could hold the line, but two servers could read the same four players and start two rounds with them. A MemoryStore **queue** hands each item to one reader at a time.

## Add, read, remove

```luau
local MemoryStoreService = game:GetService("MemoryStoreService")

-- items a server reads stay hidden from other readers for 30 seconds
local queue = MemoryStoreService:GetQueue("Matchmaking", 30)

-- a lobby server adds a waiting player, for at most 5 minutes
pcall(queue.AddAsync, queue, player.User.Id, 300)

-- a matchmaker takes four players, or none, without waiting
local ok, userIds, readId = pcall(queue.ReadAsync, queue, 4, true, 0)
if ok and #userIds == 4 then
	startMatch(userIds)
	pcall(queue.RemoveAsync, queue, readId)
end
```

`ReadAsync` returns the items and an id for the batch, but does not delete them. They turn invisible to other reads for the timeout given to `GetQueue`, 30 seconds by default. `RemoveAsync(readId)` deletes them once they are handled.

`AddAsync(value, expiration, priority)` puts higher priorities first, and equal priorities in the order added. With `allOrNothing` true, `ReadAsync` returns nothing unless it can return `count` items.

The third argument, `waitTimeout`, defaults to `-1`: wait until enough items arrive, checking every two seconds, and every check spends a request unit. Pass `0` to return at once, or a short wait.

## When a batch fails

Say the round cannot start because the teleport failed. Leave the batch alone: do not remove it, and never add the players again. Their items come back when the timeout ends, and a player added again would stand in line twice.

Make the invisibility timeout longer than a batch takes to handle. If it runs out first, another server can read the same players, and a late `RemoveAsync` does nothing.

```luau
if not startMatch(userIds) then
	-- wrong: they are still in the queue, so this adds a second copy of each
	-- for _, userId in userIds do queue:AddAsync(userId, 300) end
	return false -- right: the batch comes back after the invisibility timeout
end
pcall(queue.RemoveAsync, queue, readId)
```

Remove only after the batch is handled. Removing first and then failing loses the players; adding them again duplicates them.

**Key points**

- `GetQueue(name, invisibilityTimeout)`: the timeout defaults to 30 seconds.
- `ReadAsync(count, allOrNothing, waitTimeout)` hides the items it returns and gives an id for the batch.
- Call `RemoveAsync(id)` only after the batch is handled.
- An item that is not removed comes back after the timeout, so never add it again.
- Make the timeout longer than a batch takes, and pass a short `waitTimeout`: the default waits forever.
