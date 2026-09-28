# Hash maps, quotas and choosing a store

*The third MemoryStore structure, the quotas every structure shares, and which store fits which need.*

Roblox Advanced · Across servers · 9 min

---

A trading post lists 50,000 items, and every server reads and writes listings all the time. A sorted map could hold them, but a sorted map lives on a single partition, and every request to it counts against that one partition's limit. At this size and rate, one partition is the bottleneck.

The third MemoryStore structure spreads the load.

## Hash maps

```luau
local listings = MemoryStoreService:GetHashMap("Listings")

pcall(listings.SetAsync, listings, listingId, { item = "Sword", price = 250 }, 3600)

local ok, listing = pcall(listings.GetAsync, listings, listingId)

-- mark it sold, unless another server already did
local okSold, sold = pcall(listings.UpdateAsync, listings, listingId, function(current)
	if current == nil or current.sold then
		return nil -- gone, or already sold: cancel
	end
	current.sold = true
	return current
end, 3600)
```

The same calls as a sorted map, without sort keys and without order: there is no `GetRangeAsync`. `ListItemsAsync(count)` pages through every item when you need them all.

A hash map spreads its keys over many partitions, so traffic spread over many keys rarely nears any one partition's limit. A sorted map or a queue lives on one partition, and holds at most 1,000,000 items and 100 MB. The docs recommend a hash map past about 1,000 keys, and a sorted map for fewer. The price of a hash map is order: it cannot return its items sorted.

## Quotas

Every structure draws on two quotas shared by the whole experience:

- memory: (64 + 1.2 × concurrent users) KB. Once it is full, any request that would add memory fails.
- requests: (1,000 + 120 × concurrent users) request units per minute. Most calls cost one; `GetRangeAsync` and `ReadAsync` cost one per item returned.

The memory quota grows as soon as users join, but shrinks only days after they leave. Delete what you are done with, and keep expirations short as a safety net.

```luau
local parties = MemoryStoreService:GetHashMap("Parties")

-- the lobby stores a party for its teleport, with a 10-minute safety net
pcall(parties.SetAsync, parties, partyId, memberIds, 600)

-- the destination server, once the party has arrived
pcall(parties.RemoveAsync, parties, partyId) -- frees the memory now
```

Module 4 carries a party through a teleport this way.

| Store | Keeps data | Shape | Fits |
| --- | --- | --- | --- |
| DataStore | For good, with versions | One value per key, cached reads, per-server budgets | Profiles, purchases, anything that must survive |
| OrderedDataStore | For good, no versions | Integers, read in sorted pages | All-time leaderboards |
| MemoryStore | Until it expires, 45 days at most; never backed up | Sorted maps, queues and hash maps; fast | Server lists, matchmaking, live boards, data in transit |
| MessagingService | Nothing: a message is delivered or lost | Best effort, typically 1–2 seconds | Announcements, and requests that can be repeated |

**Key points**

- `GetHashMap(name)`: keys and values with no order, spread over many partitions. Use it past about 1,000 keys.
- A sorted map or queue lives on one partition, and holds at most 1,000,000 items and 100 MB.
- Memory and request units are quotas for the whole experience that grow with concurrent users. A full memory quota fails every request that adds memory.
- Remove items when you are done, and keep expirations short.
- Permanent: DataStore, or OrderedDataStore for sorted integers. Shared and short-lived: MemoryStore. Sent, not stored: MessagingService.
