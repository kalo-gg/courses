# Limits and retries

*What one server may spend, what one key may hold, and a retry that backs off instead of hammering a service that said no.*

Roblox Advanced · DataStores · 11 min

---

A script saves a player's key every time they pick up a coin. In a busy round, ten players pick up a coin a second each: 600 writes a minute. This server may send 460 `StandardWrite` requests a minute (60 + 40 × 10 players). Past that, requests wait in a queue, the scripts that sent them yield, and once the queue is full new requests fail.

Every call spends from a budget, and any call can also fail for other reasons.

## Budgets

```luau
local DataStoreService = game:GetService("DataStoreService")

local reads = DataStoreService:GetRequestBudgetForRequestType(Enum.DataStoreRequestType.StandardRead)
local writes = DataStoreService:GetRequestBudgetForRequestType(Enum.DataStoreRequestType.StandardWrite)
print(reads, writes) -- how many of each this server can send right now
```

`GetAsync` spends `StandardRead`, `SetAsync` spends `StandardWrite`, and `UpdateAsync` spends one of each. A read served from the cache spends nothing.

| Limit | Value |
| --- | --- |
| This server, `StandardRead` | (60 + 40 × players in this server) requests per minute |
| This server, `StandardWrite` | (60 + 40 × players in this server) requests per minute |
| Every server together, `StandardRead` (shared with Open Cloud) | (300 + 40 × concurrent users) requests per minute |
| Every server together, `StandardWrite` (shared with Open Cloud) | (300 + 20 × concurrent users) requests per minute |
| Data read from one key | 25 MB per minute per key |
| Data written to one key | 4 MB per minute per key |
| One value | 4,194,304 characters per key |
| All the experience's data | (500 + 1 × lifetime users) MB |

From the docs' limits page when this course was checked. Limits change, so the course reference links each one to its page.

## When a call fails

Calls also fail for reasons that have nothing to do with your code: a server error, a moment of throttling. Most succeed a little later. So retry, but back off: wait a second, then two, then four, up to a cap, and give up after a few attempts. Retrying at once only adds load to a service that is already turning you away. The docs' player data guide also adds random jitter to each wait, so servers that failed together do not retry together.

One failure needs extra care. A write that errored means only that no success reached your server: the docs warn the write may have landed anyway. Writing a whole profile again is harmless, but adding 50 coins twice is not. Before you repeat a write like that, read the key with `UseCache = false` and decide from what is really stored.

```luau
local ok = pcall(store.UpdateAsync, store, key, addFifty)
if not ok then
	-- it may have landed anyway: ask the backend, not the cache
	local fresh = Instance.new("DataStoreGetOptions")
	fresh.UseCache = false
	local readOk, data = pcall(store.GetAsync, store, key, fresh)
	if readOk then
		-- decide from data whether to try again
	end
end
```

The docs warn that a cached read can return stale data here, which is why the check skips the cache.

> **Recognising older code**
>
> `GetRequestBudgetForRequestType(Enum.DataStoreRequestType.GetAsync)` is older code. The enum reference says its method-named items (`GetAsync`, `SetIncrementAsync`, `UpdateAsync` and the rest) should not be used. Pass `StandardRead`, `StandardWrite`, `StandardList`, `StandardRemove` or their `Ordered` equivalents.

**Key points**

- Each server has a per-minute budget for each request type, such as `StandardWrite`: (60 + 40 × players in this server) requests per minute.
- Past the budget, requests queue and yield, and a full queue fails them.
- Retry failures with exponential backoff, a cap on the wait, and a limited number of attempts.
- A failed write may still have landed: read with `UseCache = false` before you repeat a write that is not safe to do twice.
