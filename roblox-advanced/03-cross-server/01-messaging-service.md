# MessagingService

*Publish a message on a topic, and every server subscribed to it receives it, usually within a second or two, but not always.*

Roblox Advanced · Across servers · 10 min

---

An admin types `/announce Double coins for ten minutes`. The server fires a RemoteEvent, and the thirty players on that server see it. Players on the other 200 servers see nothing: a RemoteEvent reaches only its own server's clients.

Servers of one experience talk through **MessagingService**: one server publishes a message on a topic, and every server subscribed to that topic receives it.

## Publish and subscribe

```luau
local MessagingService = game:GetService("MessagingService")

local ok, err = pcall(MessagingService.SubscribeAsync, MessagingService, "Announce", function(message)
	print("Announcement:", message.Data, "sent at", message.Sent)
end)
if not ok then
	warn("Could not subscribe:", err)
end

-- later, on any server
pcall(MessagingService.PublishAsync, MessagingService, "Announce", "Double coins for ten minutes")
```

Each message arrives as a table: `Data` is what was published, and `Sent` is when, in Unix seconds. `SubscribeAsync` returns a connection; `Disconnect` it to stop listening. Both calls are web requests, so both go in `pcall`.

A topic is any string of up to 80 characters. Use one fixed topic for a broadcast, such as `Announce`. To reach one server, give each server its own topic, built from `game.JobId`.

A message holds at most 1 kB, so send a short string or an id, never a profile. One server may send (600 + 240 × players in this server) messages per minute per server, and topics and subscriptions have caps too.

**Asking the owner of a lock to let go**

```luau
-- every server listens on a topic of its own
pcall(MessagingService.SubscribeAsync, MessagingService, "Release_" .. game.JobId, function(message)
	releaseSession(message.Data) -- save, then clear the lock
end)

-- a server that finds the key locked asks the owner the lock names
local owner = keyInfo:GetMetadata().JobId
pcall(MessagingService.PublishAsync, MessagingService, "Release_" .. owner, key)
```

Module 2's lock records its owner's `JobId`, so the request reaches that one server. The claim still retries with backoff and still steals a stale lock: the message only shortens the wait.

## Best effort

Delivery typically takes 1 to 2 seconds, and the docs promise no more: it is best effort, not guaranteed. `PublishAsync` returns once the backend has the message, not once any server has it.

Design as if any message can be lost. A message is a nudge; the state lives in a DataStore or MemoryStore. A lost release request only means the claim waits for its next retry, or for the lock to go stale.

> **Recognising older code**
>
> `store:OnUpdate(key, callback)` let a server watch a DataStore key for other servers' writes. It polled for them only once a minute, and it is deprecated: the docs say to publish the change with MessagingService instead.

**Key points**

- `PublishAsync(topic, message)` reaches every server subscribed to the topic, the sender included.
- Messages arrive as `{ Data, Sent }`. Keep them small: a message holds at most 1 kB.
- Use a fixed topic for broadcasts, and a topic built from `game.JobId` to reach one server.
- Delivery is best effort. Keep the state in a store, and treat a message as a nudge that may be lost.
- Both calls are web requests: wrap them in `pcall`.
