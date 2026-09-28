# Rate limits

*A valid request can still arrive too often. Limit each player with a cooldown or a token bucket, drop the excess, and forget them when they leave.*

Roblox Intermediate · Trust · 11 min

---

A `Cheer` remote plays a cheer for everyone: the server receives it and calls `FireAllClients`. It takes no arguments, so there is nothing to validate. Then an exploiter fires it hundreds of times a second, and every player in the server hears hundreds of cheers a second.

Each request was valid. The rate was the attack, and only the server can limit it.

## A cooldown per player

```luau
local COOLDOWN = 0.5
local lastCheer: { [Player]: number } = {}

Cheer.OnServerEvent:Connect(function(player)
	local now = os.clock()
	if now - (lastCheer[player] or -math.huge) < COOLDOWN then
		return -- too soon: drop it
	end
	lastCheer[player] = now
	Cheer:FireAllClients(player)
end)
```

`os.clock()` measures elapsed seconds and never jumps when the system clock is adjusted. The state is keyed by the player and lives on the server, where the client cannot reset it.

## A token bucket

A fixed cooldown is strict: a double-click or a quick combo loses its second press. The docs' security guide uses a **token bucket** instead. Each player's bucket holds up to `CAPACITY` tokens and refills at a steady rate. Each request spends one, and a request that finds less than one token is dropped. Short bursts get through, and the average cannot rise above the refill rate.

```luau
local CAPACITY = 5 -- the biggest burst
local REFILL = 5 -- tokens per second
local buckets: { [Player]: { tokens: number, last: number } } = {}

local function allow(player: Player): boolean
	local now = os.clock()
	local bucket = buckets[player]
	if bucket == nil then
		bucket = { tokens = CAPACITY, last = now }
		buckets[player] = bucket
	end
	bucket.tokens = math.min(CAPACITY, bucket.tokens + (now - bucket.last) * REFILL)
	bucket.last = now
	if bucket.tokens < 1 then
		return false
	end
	bucket.tokens -= 1
	return true
end
```

The bucket refills lazily: nothing runs between requests, and each call adds what the time since the last one has earned, up to `CAPACITY`.

When a request is over the limit, drop it and return. Lag can bunch an honest player's messages together, so kicking at the first overflow punishes the wrong people. Save `player:Kick()` for rates no real client would produce.

The buckets are keyed by the player, so clear the entry on `PlayerRemoving`, as the docs' example does. Otherwise every player who ever joined stays in the table.

> **Limit on the client too**
>
> Disable a button for its cooldown, so honest players never hit the server's limit. That is for their comfort, not for security: the docs say never to rely on a client-side limit alone.

**Key points**

- A valid request can still be abuse: limit how often each player can make it, on the server.
- Keep limiter state per player, keyed by the Player, and time it with `os.clock()`.
- A cooldown allows one request per interval; a token bucket allows bursts up to its capacity and caps the average at its refill rate.
- Drop requests over the limit quietly; kick only for rates no honest client could reach.
- Clear the player's entry on `PlayerRemoving`.
