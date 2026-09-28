# Roblox Advanced reference

*Roblox Advanced 0.740.19.7400931, released September 22, 2026. Checked on September 27, 2026.*

---

## DataStoreService, GlobalDataStore and OrderedDataStore

DataStores keep data across servers and sessions. Every call is a web request that can fail or be throttled, so each one goes in a `pcall`, and the budgets in the Limits group apply.

### `GetDataStore`

```luau
DataStoreService:GetDataStore(name: string, scope: string?, options: DataStoreOptions?) -> DataStore
```

Returns the DataStore with that name; the same name and scope always return the same object. Only server scripts can use DataStoreService.

> A place tested in Studio reaches DataStores only after **Enable Studio Access to API Services** is on in Experience Settings ⟩ Security, and then it reads and writes the same data as the live game. The docs advise enabling it only on a separate test experience.

### `Player.User`

```luau
Player.User: User  -- read-only
User.Id: number
User:ToString() -> string
("player_%d"):format(player.User.Id)  -- the users guide's key
```

A player's identity in this experience. The users guide calls it the standard identifier for new code, builds DataStore keys from `player.User.Id`, and serialises a `User` with `ToString` when other systems must read it. Engine APIs that take a user ID accept a `User` too.

> Roblox is rolling out domain-scoped IDs: a player new to an experience gets an ID unique to that experience. A player who joined before the rollout keeps their global ID there, so existing keys built from `Player.UserId` keep working. `UserId` is not deprecated.

### `GetAsync`

```luau
GlobalDataStore:GetAsync(key: string, options: DataStoreGetOptions?) -> (any, DataStoreKeyInfo?)  -- yields
```

The key's latest value and its `DataStoreKeyInfo`, or `nil, nil` if the key does not exist or was removed. A read is cached for a few seconds (see Limits), and a write from this server updates the cache at once.

> To read what the backend holds, such as after a write that errored, pass a `DataStoreGetOptions` with `UseCache = false`. That read always counts against the budgets.

### `SetAsync`

```luau
GlobalDataStore:SetAsync(key: string, value: any, userIds: {number}?, options: DataStoreSetOptions?) -> string  -- yields
```

Writes the value, the user IDs and the metadata, and returns the new version's identifier. It spends only the write budget, but it does not read first, so the docs warn of inconsistent data when two servers set the same key at once.

> Strings must be valid UTF-8. The docs say metadata must be set on every write, even when unchanged, or the current metadata is lost. They also recommend passing `userIds` to help with GDPR removal.

### `UpdateAsync`

```luau
GlobalDataStore:UpdateAsync(
  key: string,
  transform: (current: any, keyInfo: DataStoreKeyInfo?) -> (any?, {number}?, {[string]: any}?)
) -> (any, DataStoreKeyInfo)  -- yields
```

Reads the latest value, passes it to `transform`, and writes what `transform` returns. If another server writes the key between the read and the write, the engine calls `transform` again with the newer value and discards the earlier result. Returning `nil` cancels the write. One call spends both a read and a write.

> `transform` must not yield. Its second and third results are the user IDs and metadata: return `keyInfo:GetUserIds()` and `keyInfo:GetMetadata()` unless you are changing them, or they are cleared.

### `IncrementAsync, RemoveAsync`

```luau
GlobalDataStore:IncrementAsync(key: string, delta: number?, userIds: {number}?, options: DataStoreIncrementOptions?) -> number  -- yields
GlobalDataStore:RemoveAsync(key: string) -> (any, DataStoreKeyInfo?)  -- yields
```

`IncrementAsync` adds an integer `delta` (default 1) to an integer value. `RemoveAsync` returns the value it removed and writes a tombstone version, so `GetAsync` returns `nil` while older versions stay readable until they expire.

> Removed objects are deleted permanently after 30 days. An OrderedDataStore has no versions, so a key removed there is gone at once.

### `DataStoreKeyInfo`

```luau
DataStoreKeyInfo.Version: string
DataStoreKeyInfo.CreatedTime: number  -- ms since epoch
DataStoreKeyInfo.UpdatedTime: number
DataStoreKeyInfo:GetUserIds() -> {number}
DataStoreKeyInfo:GetMetadata() -> {[string]: any}
```

Describes one version of a key. `GetAsync`, `UpdateAsync`, `IncrementAsync`, `RemoveAsync` and `GetVersionAsync` return it, and `UpdateAsync` passes it to the transform.

### `Right to be forgotten`

```luau
-- a deletion template, set in the Data Stores Manager or the Open Cloud Configs API
key_pattern = "User_{UserId}"
```

When Roblox processes a right-to-erasure request, it deletes the keys and stores that match your deletion templates, with `{UserId}` replaced by the requester's ID. For a key to qualify, the user ID must appear in the store's name or scope or the key's name, in a fixed pattern.

> A key built as `"User_" .. id` fits the pattern `User_{UserId}`. An experience can have up to 100 templates.

### `Options objects`

```luau
Instance.new("DataStoreGetOptions").UseCache: boolean
Instance.new("DataStoreSetOptions"):SetMetadata(attributes: {[string]: any})
Instance.new("DataStoreIncrementOptions"):SetMetadata(attributes: {[string]: any})
Instance.new("DataStoreOptions").AllScopes: boolean
```

Extra arguments, passed as instances. `DataStoreGetOptions` can skip the cache; the set and increment options carry custom metadata for the key; `DataStoreOptions`, passed to `GetDataStore`, lists keys across every scope.

> Metadata keys may be 50 characters and values 250, with all pairs together at most 300 characters.

### `ListVersionsAsync, GetVersionAsync, GetVersionAtTimeAsync`

```luau
DataStore:ListVersionsAsync(key: string, sortDirection: Enum.SortDirection?, minDate: number?, maxDate: number?, pageSize: number?) -> DataStoreVersionPages  -- yields
DataStore:GetVersionAsync(key: string, version: string) -> (any, DataStoreKeyInfo)  -- yields
DataStore:GetVersionAtTimeAsync(key: string, timestamp: number) -> (any, DataStoreKeyInfo?)  -- yields
```

Versions for rollback. The first write to a key in each UTC hour is kept as a backup; later writes in the same hour overwrite it. A backup expires 30 days after a newer write replaces it, and the latest version never expires. `GetVersionAtTimeAsync` returns the version that was current at a Unix time in milliseconds, or `nil` if none was.

> The timestamp must be above zero and no more than ten minutes in the future. To roll back, read the old version and write it again with `SetAsync`, passing its metadata.

### `OrderedDataStore`

```luau
DataStoreService:GetOrderedDataStore(name: string, scope: string?) -> OrderedDataStore
OrderedDataStore:GetSortedAsync(ascending: boolean, pageSize: number, minValue: number?, maxValue: number?) -> DataStorePages  -- yields
```

A DataStore whose values must be integers, readable in sorted order. `GetSortedAsync` returns pages whose `GetCurrentPage()` is an array of `{ key, value }` tables; `AdvanceToNextPageAsync()` moves on until `IsFinished` is true.

> `pageSize` may be at most 100. Ordered stores have no versions, metadata or user IDs, so their `DataStoreKeyInfo` is always `nil`, and they have their own, smaller budgets.

### `Request budgets`

```luau
DataStoreService:GetRequestBudgetForRequestType(requestType: Enum.DataStoreRequestType) -> number
DataStoreService:SetRateLimitForRequestType(requestType: Enum.DataStoreRequestType, baseLimit: number, perPlayerLimit: number)
```

How many requests of a type this server can make right now, and a way to change this server's per-minute limit to `baseLimit + perPlayerLimit × players`. Requests past the budget are throttled: they queue, yielding the caller, and a full queue fails them.

> Pass the modern request types (`StandardRead`, `StandardWrite`, `StandardList`, `StandardRemove` and the `Ordered` four). The enum reference says its older items, such as `GetAsync` and `SetIncrementAsync`, should not be used. The docs recommend calling `SetRateLimitForRequestType` once per type at startup.

### `Failures`

```luau
local ok, result = pcall(store.UpdateAsync, store, key, transform)
```

Any DataStore call can throw. A failed write means only that the server got no success response: the docs warn that the write may still have landed, so check with an uncached read before deciding. Each request queue holds 30 requests, after which new ones fail with errors 301 to 306.

> The player data guide retries with exponential backoff, jitter and a cap, and keeps each key's requests in order, so an old retry cannot overwrite a newer write.

### `BindToClose`

```luau
game:BindToClose(fn: (reason: Enum.CloseReason) -> ())
Enum.CloseReason: Unknown | RobloxMaintenance | DeveloperShutdown | DeveloperUpdate | ServerEmpty | OutOfMemory | Moderation
```

Runs `fn` before the server shuts down. Every bound function runs at the same time, and the server waits for them only for the window in Limits. It is where unsaved data is saved when the server closes with players still in it.

> The docs suggest `RunService:IsStudio()` to skip the bound work in Studio test sessions. `DeveloperShutdown` is also the reason given when Studio calls them. The deprecated `game.OnClose` callback is the old form of this hook.

## MemoryStoreService: sorted maps, queues and hash maps

Fast, shared, short-lived data. Every item expires, nothing is backed up, and every structure is shared by every server in the experience.

### `Getting a structure`

```luau
MemoryStoreService:GetSortedMap(name: string) -> MemoryStoreSortedMap
MemoryStoreService:GetQueue(name: string, invisibilityTimeout: number?) -> MemoryStoreQueue  -- default 30 s
MemoryStoreService:GetHashMap(name: string) -> MemoryStoreHashMap
```

The name is global within the experience, so every server that uses the same name gets the same structure. Every call on a structure is a network request, so the guides wrap each one in `pcall`.

> The place must be published. Studio sees its own data, never production's, but under the same quotas, which are tiny with one user. The 0.740 dump also has `GetDistributedCounter`, which the engine reference does not document yet.

### `Sorted map: SetAsync, GetAsync`

```luau
MemoryStoreSortedMap:SetAsync(key: string, value: any, expiration: number, sortKey: (number | string)?) -> boolean  -- yields
MemoryStoreSortedMap:GetAsync(key: string) -> (any?, (number | string)?)  -- yields
```

`SetAsync` adds or overwrites a key, returning `true` if it overwrote one; the item is removed after `expiration` seconds. Items sort by sort key first, then by key: numeric sort keys, then string sort keys, then items with none.

> Keys and sort keys may be 128 characters and values 32 KB.

### `Sorted map: GetRangeAsync`

```luau
MemoryStoreSortedMap:GetRangeAsync(
  direction: Enum.SortDirection,
  count: number,  -- at most 200
  exclusiveLowerBound: { key: string?, sortKey: any? }?,
  exclusiveUpperBound: { key: string?, sortKey: any? }?
) -> { { key: string, value: any, sortKey: any? } }  -- yields
```

Up to `count` items in order, between two exclusive bounds. It costs one request unit per item returned.

### `Sorted map: UpdateAsync, RemoveAsync`

```luau
MemoryStoreSortedMap:UpdateAsync(key: string, transform: (value: any?, sortKey: any?) -> (any?, any?), expiration: number) -> (any, any)  -- yields
MemoryStoreSortedMap:RemoveAsync(key: string)  -- yields
MemoryStoreSortedMap:GetSizeAsync() -> number  -- yields
```

`UpdateAsync` passes the current value and sort key (`nil` for a new key) to `transform` and saves what it returns only if nobody changed the item meanwhile; otherwise it calls `transform` again. Returning `nil` cancels. After too many retries it fails with a conflict.

### `Queue: AddAsync, ReadAsync`

```luau
MemoryStoreQueue:AddAsync(value: any, expiration: number, priority: number?)  -- yields; priority default 0
MemoryStoreQueue:ReadAsync(count: number, allOrNothing: boolean?, waitTimeout: number?) -> ({any}, string)  -- yields
```

Higher priorities are read first, and equal priorities in the order added. `ReadAsync` returns up to `count` items (at most 100) and an id for removing them. The items are not deleted: they turn invisible to other reads for the invisibility timeout set in `GetQueue`.

> With `allOrNothing` true it returns nothing unless `count` items are there. `waitTimeout` defaults to `-1`, which waits forever, polling every two seconds and spending a request unit each time; pass `0` to return at once.

### `Queue: RemoveAsync`

```luau
MemoryStoreQueue:RemoveAsync(id: string)  -- yields
MemoryStoreQueue:GetSizeAsync(excludeInvisible: boolean?) -> number  -- yields
```

Deletes the items a `ReadAsync` returned, once they are handled. Items that are never removed become visible again when the invisibility timeout ends, so a failed batch is retried without being added again.

> Called after the timeout has passed, `RemoveAsync` has no effect. `GetSizeAsync` counts invisible items unless `excludeInvisible` is true.

### `Hash map`

```luau
MemoryStoreHashMap:SetAsync(key: string, value: any, expiration: number) -> boolean  -- yields
MemoryStoreHashMap:GetAsync(key: string) -> any?  -- yields
MemoryStoreHashMap:UpdateAsync(key: string, transform: (value: any?) -> any?, expiration: number) -> any  -- yields
MemoryStoreHashMap:RemoveAsync(key: string)  -- yields
MemoryStoreHashMap:ListItemsAsync(count: number) -> MemoryStoreHashMapPages  -- yields
```

Keys and values with no order, spread across many partitions, so traffic on many keys rarely hits a partition limit. The docs suggest a hash map past about 1,000 keys and a sorted map below that. `UpdateAsync` works like the sorted map's, without a sort key.

> Keys may be 128 characters and values 32 KB. `ListItemsAsync` takes 1 to 200 and costs a unit per partition scanned plus one per item.

### `Expiration and deletion`

```luau
-- every write takes an expiration in seconds
-- RemoveAsync frees the memory at once
```

Items vanish when they expire, which frees quota. The docs recommend deleting items explicitly when you are done with them and keeping expiration short as a safety net. Once the memory quota is full, writes that add memory fail until items expire or are removed.

## MessagingService

Publish and subscribe between the servers of one experience, by topic. Fast, but best effort.

### `PublishAsync`

```luau
MessagingService:PublishAsync(topic: string, message: any)  -- yields
```

Sends `message` to every server subscribed to `topic`. It yields until the backend has received it, not until anyone has.

### `SubscribeAsync`

```luau
MessagingService:SubscribeAsync(topic: string, callback: (message: { Data: any, Sent: number }) -> ()) -> RBXScriptConnection  -- yields
```

Calls `callback` for each message on the topic. `Data` is what was published; `Sent` is the Unix time in seconds it was sent. It yields until the subscription is registered, and may be called more than once for the same topic.

> Call `Disconnect` on the connection to unsubscribe. Stopping the script that holds the connection does the same.

### `Delivery`

```luau
-- typically within 1–2 seconds
-- best effort, not guaranteed
```

The class page says delivery usually takes a second or two and is not guaranteed, so design as if a message can be lost. It suits announcements and requests that can be repeated; the state itself belongs in a DataStore or MemoryStore.

> The page also caps messages received per topic and across the experience, and subscriptions per server; the Limits group has all its numbers.

## MarketplaceService and BadgeService

Developer products are granted by a receipt handler that answers once the grant is saved. Passes and badges are owned once.

### `PromptProductPurchase`

```luau
MarketplaceService:PromptProductPurchase(player: Player, productId: number)
```

Shows the purchase prompt for a developer product. The purchase itself arrives later, as a receipt, on whatever server the buyer is in.

> The docs say never to grant a product from `PromptProductPurchaseFinished`: its firing does not mean the purchase went through.

### `BindReceiptHandler`

```luau
MarketplaceService:BindReceiptHandler(
  transactionType: Enum.ReceiptType,  -- DeveloperProduct | RobuxTransferSender | RobuxTransferReceiver
  handler: (receiptInfo: {[string]: any}) -> Enum.ReceiptDecision,
  filter: {number}?  -- developer product IDs
) -> RBXScriptConnection
```

Registers a receipt handler. With a `filter`, it receives only receipts for those product IDs; without one it is the catch-all for every product no filtered handler claims. Disconnect the connection to unbind it.

> It errors if a handler is already bound for the same type and product, or a second catch-all is bound. Filters work only for developer products.

### `Receipt info`

```luau
receiptInfo.PurchaseId: string
receiptInfo.PlayerId: number
receiptInfo.ProductId: number
receiptInfo.CurrencySpent: number
receiptInfo.PlaceIdWherePurchased: number
receiptInfo.ReceiptType: Enum.ReceiptType
```

`PurchaseId` is unique to the receipt, which makes it the key for granting exactly once: store it in the buyer's saved data with the grant, and answer `Processed` straight away when it is already there.

### `ReceiptDecision`

```luau
Enum.ReceiptDecision.Processed
Enum.ReceiptDecision.NotProcessedYet
```

`Processed` means everything was granted and saved; the receipt is complete and never delivered again. `NotProcessedYet` leaves it unresolved, and so does a buyer who is not reachable: it is delivered again later, in the same session or the next time the buyer joins a server.

> The buyer must be on a server for a handler to run. The docs name no retry interval for bound handlers.

### `ProcessReceipt (legacy)`

```luau
MarketplaceService.ProcessReceipt = function(receiptInfo): Enum.ProductPurchaseDecision
```

The older single callback for every product, returning `PurchaseGranted` or `NotProcessedYet`. The `BindReceiptHandler` page calls it legacy. A developer product receipt that no bound handler matches still falls through to it, so the two can coexist while a game migrates.

> Its page records behaviour worth designing for with either API: no timed retry (only the buyer's next purchase or rejoin), and the same receipt running on two servers at once when the buyer joins a second server before the first returns.

### `UserOwnsGamePassAsync`

```luau
MarketplaceService:UserOwnsGamePassAsync(userId: number | User, gamePassId: number) -> boolean  -- yields
```

Whether the user owns the pass. The result is cached, and `PromptGamePassPurchaseFinished` updates the cache; on a user's first entry to a server after buying the pass it always returns true.

> A pass bought outside the experience mid-session can take several minutes to show. The ID is a pass ID, not an asset ID.

### `PromptGamePassPurchase, PromptGamePassPurchaseFinished`

```luau
MarketplaceService:PromptGamePassPurchase(player: Player, gamePassId: number)
MarketplaceService.PromptGamePassPurchaseFinished(player: Player, gamePassId: number, wasPurchased: boolean)  -- event
```

The event fires when the pass prompt closes, whether the player bought, cancelled or saw an error. The passes guide grants the pass there when `wasPurchased` is true, and checks `UserOwnsGamePassAsync` on join for players who already own it.

> Roblox does not record which user bought a pass for you; store it yourself if you need the history.

### `GetProductInfoAsync`

```luau
MarketplaceService:GetProductInfoAsync(assetId: number, infoType: Enum.InfoType?) -> {[string]: any}  -- yields; default Asset
```

Name, description, price and more for an asset, developer product (`Enum.InfoType.Product`), pass (`GamePass`), subscription or bundle. It throws if no item has that ID.

### `Subscriptions`

```luau
MarketplaceService:PromptSubscriptionPurchase(user: Player, subscriptionId: string)
MarketplaceService:GetUserSubscriptionStatusAsync(user: Player, subscriptionId: string) -> { IsSubscribed: boolean, IsRenewing: boolean }  -- yields
Players.UserSubscriptionStatusChanged(user: Player, subscriptionId: string)  -- event, server
```

`IsSubscribed` is true only once payment has gone through. The server checks for a change only after the subscription prompt closes, so a player who subscribed outside the experience must still be prompted before the event fires.

### `AwardBadgeAsync`

```luau
BadgeService:AwardBadgeAsync(userId: number | User, badgeId: number) -> boolean  -- yields
```

Awards a badge from a server script. The player must be in the experience and not already own the badge, and the badge must be enabled and belong to this experience.

> A player who deletes a badge from their profile can be awarded it again. The rate limit is (50 + 35 × users) requests per minute.

### `UserHasBadgeAsync, CheckUserBadgesAsync`

```luau
BadgeService:UserHasBadgeAsync(userId: number | User, badgeId: number) -> boolean  -- yields
BadgeService:CheckUserBadgesAsync(userId: number | User, badgeIds: {number}) -> {number}  -- yields
```

Whether a user owns one badge, or which of up to 10 badges they own. From a client script they work only for the local player.

## TeleportService and TeleportOptions

Teleports move players between places and servers, only from the server, and only in a published experience: they do not work in Studio playtests.

### `TeleportAsync`

```luau
TeleportService:TeleportAsync(placeId: number, players: {Player}, teleportOptions: TeleportOptions?) -> TeleportAsyncResult?  -- yields
```

The one teleport method: to a place, to a specific server, or to a reserved server. A group must stay within one experience, and one call can carry a limited number of players (see Limits). With options passed, it returns a `TeleportAsyncResult` naming where they went.

> It throws for bad arguments and network failures, so call it in `pcall` and retry; the guide's `SafeTeleport` retries a few times with a delay. It errors on the client.

### `TeleportOptions`

```luau
local options = Instance.new("TeleportOptions")
options.ShouldReserveServer: boolean
options.ReservedServerAccessCode: string
options.ServerInstanceId: string  -- a JobId
```

Where to send the players. `ShouldReserveServer` creates a new reserved server; `ReservedServerAccessCode` joins an existing one; `ServerInstanceId` joins a public server. With none set they go to a public server, matched on the first player in the list.

> `ShouldReserveServer` cannot be combined with either of the other two, and the access code cannot be combined with `ServerInstanceId`.

### `ReserveServerAsync`

```luau
TeleportService:ReserveServerAsync(placeId: number) -> (string, string)  -- yields: access code, PrivateServerId
```

Reserves a server for a place without teleporting anyone, returning an access code for `ReservedServerAccessCode`. The server starts when the code is first used, and the code stays valid indefinitely.

> A reserved server has a non-empty `game.PrivateServerId` and a `game.PrivateServerOwnerId` of 0.

### `SetTeleportData`

```luau
TeleportOptions:SetTeleportData(teleportData: any)
```

Data to carry to the destination. It may hold value types only: numbers, strings, booleans, Roblox value types and tables of them. Instances, functions and SharedTables are removed, with an error in the console.

> The teleport guide says not to pass secure data this way: it travels through the client, visible and unencrypted. Send an ID and keep the real data in a DataStore or MemoryStore.

### `GetJoinData`

```luau
Player:GetJoinData() -> { SourcePlaceId: number?, SourceGameId: number?, Members: {number}?, TeleportData: any?, … }
```

How the player arrived, read on the server. Its teleport data is guaranteed to have been sent by a Roblox server within the last 48 hours, with this player, and `SourcePlaceId` and `SourceGameId` are guaranteed real, so you can check it came from one of your places.

> The same page warns that the data is still transmitted by the client and can be abused. Treat `TeleportData` as a claim to check against server-side state, never as the state.

### `TeleportInitFailed`

```luau
TeleportService.TeleportInitFailed(player: Player, teleportResult: Enum.TeleportResult, errorMessage: string, placeId: number, teleportOptions: TeleportOptions)  -- event
```

Fires when a teleport that started fails later and the player stays in this server; `TeleportAsync` has already returned by then. It fires once per player in a group. The `teleportOptions` it passes is a new object that sends the player to the same destination, ready to retry.

> The guide waits longer before retrying a `Flooded` result, and reports rather than retries results that mean the teleport is invalid.

## Actor, SharedTable and thread safety

Scripts under different Actors can run on different cores in the parallel phase. They may read the world there, but must return to the serial phase to change it.

### `Actor`

```luau
Instance.new("Actor")  -- a Model; put the scripts and their instances inside
script:GetActor() -> Actor?
```

A unit of parallel execution: the scripts under one Actor always run one after another, so parallel work needs several Actors. The guide suggests many (it gives 64 for hit validation on a 4-core machine) but split by logic, not at random.

> Each Actor runs its own Luau VM, so a ModuleScript required under two Actors runs twice and shares no state. The guide advises against nesting Actors; a nested script belongs to its closest Actor.

### `task.desynchronize, task.synchronize`

```luau
task.desynchronize()  -- yields to the next parallel phase
task.synchronize()  -- yields to the next serial phase
```

Scripts under an Actor still start serial. `desynchronize` suspends the script until the next parallel phase, and `synchronize` until the next serial one; each returns at once if the script is already in that phase.

> Both error in a script that is not under an Actor. `require` cannot be called in the parallel phase, so require modules first.

### `ConnectParallel`

```luau
RBXScriptSignal:ConnectParallel(func: (...any) -> ()) -> RBXScriptConnection
```

Runs the handler in the parallel phase each time the signal fires, and is more efficient than connecting and then calling `task.desynchronize`. Call `task.synchronize` inside it before changing instances.

### `Thread safety`

```luau
-- Unsafe: cannot be read, written or called in parallel
-- Read Parallel: can be read, not written, in parallel
-- Local Safe: usable within the same Actor only
-- Safe: can be read, written and called
```

Every API member has one of these levels, shown on its reference page; one with no level is Unsafe. In the parallel phase the engine detects and blocks unsafe access. Most properties are Read Parallel and most methods and events are Unsafe, which is why the usual shape is read in parallel, write in serial.

> The dump spells the levels `Unsafe`, `ReadSafe` and `Safe`. No member in dump 0.740 carries Local Safe.

### `SendMessage, BindToMessage, BindToMessageParallel`

```luau
Actor:SendMessage(topic: string, ...: any)
Actor:BindToMessage(topic: string, fn: (...any) -> ()) -> RBXScriptConnection
Actor:BindToMessageParallel(topic: string, fn: (...any) -> ()) -> RBXScriptConnection
```

Messages between Actors. `SendMessage` returns at once; a script under the target Actor receives the arguments in a serial or parallel callback. All three are thread-safe.

> Arguments are copied across VM boundaries, so a large table is copied in full on every send. Functions cannot be sent.

### `SharedTable`

```luau
SharedTable.new(t: {[string | number]: any}?) -> SharedTable
st[key] = value
SharedTable.size(st: SharedTable) -> number
SharedTable.clear(st: SharedTable)
```

A table every Actor can read and write, with no copying: an update from one Actor is visible to all at once. Keys are strings or non-negative integers below 2^32; values are booleans, numbers, vectors, strings, other SharedTables or serialisable data types.

> Plain tables inside `t` become SharedTables. Two SharedTables never compare equal, even with equal contents.

### `SharedTable.update, increment`

```luau
SharedTable.update(st: SharedTable, key: string | number, f: (value: any) -> any)
SharedTable.increment(st: SharedTable, key: string | number, delta: number) -> number
```

Atomic changes. Reading a value and then writing it back can interleave with another Actor, so change it with `update`, whose `f` may run more than once. `increment` returns the old value and is much faster than `update` for numbers.

### `clone, cloneAndFreeze`

```luau
SharedTable.clone(st: SharedTable, deep: boolean?) -> SharedTable
SharedTable.cloneAndFreeze(st: SharedTable, deep: boolean?) -> SharedTable
SharedTable.isFrozen(st: SharedTable) -> boolean
```

A cheap copy by structural sharing: a consistent snapshot of the table, even while others write to it. A shallow clone shares nested SharedTables; a deep one copies them, but only each table's snapshot is atomic. A frozen table is read-only and errors on writes.

### `SharedTableRegistry`

```luau
SharedTableRegistry:GetSharedTable(name: string) -> SharedTable
SharedTableRegistry:SetSharedTable(name: string, st: SharedTable?)
```

Shares a SharedTable by name with every script in the DataModel. `GetSharedTable` creates an empty one if the name is unused; `SetSharedTable` with `nil` unregisters it. The other way to share one is to send it in a message.

## Streaming properties

Tuning instance streaming, which Roblox Intermediate introduced. The Workspace properties are set in Studio; scripts cannot set them.

### `StreamingMinRadius, StreamingTargetRadius`

```luau
Workspace.StreamingMinRadius: number  -- the recommended default, 64
Workspace.StreamingTargetRadius: number  -- the recommended default, 1024
```

Around each replication focus, content inside the minimum radius streams in first and never streams out; content up to the target radius streams in as it can. Keep the target larger than the minimum: the gap is a buffer for when the client briefly stops receiving.

### `StreamingIntegrityMode`

```luau
Workspace.StreamingIntegrityMode: Enum.StreamingIntegrityMode  -- Default | Disabled | MinimumRadiusPause | PauseOutsideLoadedArea
```

What happens when a player moves into an area that has not streamed in. The guide recommends `PauseOutsideLoadedArea`, which pauses that player's gameplay until it has.

> `Player.GameplayPaused` says whether the player is paused, for a custom pause screen.

### `StreamOutBehavior, ModelStreamingBehavior`

```luau
Workspace.StreamOutBehavior: Enum.StreamOutBehavior  -- Default | LowMemory | Opportunistic
Workspace.ModelStreamingBehavior: Enum.ModelStreamingBehavior  -- Default | Legacy | Improved
```

With `LowMemory`, which the guide calls the default, content beyond the minimum radius streams out only when memory runs low; the guide recommends `Opportunistic`, which also removes content beyond the target radius without memory pressure. It also recommends `Improved` model streaming, which sends a nonatomic model that has parts only when one of them streams in, and streams it out with its last part.

### `ModelStreamingMode`

```luau
Model.ModelStreamingMode: Enum.ModelStreamingMode  -- Default | Atomic | Persistent | PersistentPerPlayer | Nonatomic
```

Per model. The `Default` value behaves as `Nonatomic`: the model's parts stream in and out on their own. `Atomic` streams the whole model in and out together. `Persistent` sends it once, soon after joining, and never streams it out. `PersistentPerPlayer` is persistent for the players added with `AddPersistentPlayer` and atomic for everyone else.

> The guide reserves `Persistent` for rare cases, such as a few parts client scripts must always reach; it is not a way around streaming. Set the mode in Studio or a server Script, never a LocalScript.

### `AddPersistentPlayer, PersistentLoaded`

```luau
Model:AddPersistentPlayer(player: Player?)
Model:RemovePersistentPlayer(player: Player?)
Model:GetPersistentPlayers() -> {Player}
Workspace.PersistentLoaded(player: Player)  -- event
```

Makes a model persistent for one player, which takes effect only when its mode is `PersistentPerPlayer`. Client code that needs persistent models should wait for `PersistentLoaded`, which fires once they have all arrived.

### `RequestStreamAroundAsync`

```luau
Player:RequestStreamAroundAsync(position: Vector3, timeOut: number?)  -- yields
```

Asks the server to stream the area around `position` to the player, before you move them there. The effect is temporary and nothing is guaranteed: client memory and the network decide what arrives.

### `ReplicationFocus, AddReplicationFocus`

```luau
Player.ReplicationFocus: Instance?
Player:AddReplicationFocus(part: BasePart)
Player:RemoveReplicationFocus(part: BasePart)
```

Streaming centres on the character's `PrimaryPart` unless `ReplicationFocus` names another part, and extra foci stream more areas. Client physics runs only in streamed areas, even for persistent models.

> Each extra focus costs the server about as much as another player moving around, so add them sparingly.

## Profiling: debug labels, the MicroProfiler and the Developer Console

Measure before you optimise. The MicroProfiler shows what ran inside each frame; the Developer Console's Script Profiler, Memory and Network tools cover the rest.

### `debug.profilebegin, debug.profileend`

```luau
debug.profilebegin(label: string)
debug.profileend()
```

Times everything between the two calls and shows it as a label on the MicroProfiler timeline. `profileend` takes no arguments: it closes the most recent label `profilebegin` opened, so labels nest.

> Wrap the part of a handler you suspect, then compare its width on the timeline with the engine's tasks in the same frame.

### `MicroProfiler`

```luau
-- Studio or the desktop client: Ctrl+F6 (⌘F6)
-- mobile: Settings ⟩ MicroProfiler On, then open the address and port it shows in a browser on the same network
```

Timing for every engine task in each frame. The bar graph along the top has one bar per frame, taller for a longer frame; the timeline below shows the tasks that ran in it, wider for longer, with each task's parts stacked under it. Find a spike in the graph, then the task that caused it in the timeline.

> It measures frame time: 16.67 ms a frame is 60 FPS, and steady frame times matter as much as short ones. The guide calls a mobile device the best place to profile, because most players use one. Ctrl+P (⌘P) pauses; the Dump menu saves frames to an HTML file. For the server, the Developer Console's MicroProfiler tool records at most 60 frames, on the desktop client in a game you can edit.

### `Script Profiler`

```luau
-- Developer Console (F9) ⟩ ScriptProfiler ⟩ Client or Server ⟩ Start
```

Records the CPU time of every running script's function calls, including method calls and property accesses, then lists them by cost: the **Callgraph** view groups them under the frame task that ran them, and **Functions** is a flat list. Choose **Client** (the default) or **Server** before you start.

> It samples 1,000 times a second by default, or 10,000 for more precision at more cost, for 1, 5 or 10 minutes or until you stop it. It records only calls that use the CPU, so a thread that waits or sleeps costs nothing there. **Export** saves a session as JSON.

### `Memory and Luau heap`

```luau
-- Developer Console (F9) ⟩ Memory, or LuauHeap ⟩ Create Snapshot
debug.setmemorycategory(tag: string) -> string
```

**Memory** shows live use by category in MB, on the client or the server; **PlaceMemory** (what you built) and **PlaceScriptMemory** (your scripts) are the ones you control. **Luau heap** takes snapshots of script memory, broken down by type, engine class or memory category. Compare two snapshots to find growth, and check **Unique References** and **Unparented Instances** for instances outside the DataModel that scripts still hold.

> A memory category is named after its script unless `debug.setmemorycategory` names the current thread's, which keeps two systems in one script apart.

### `Network`

```luau
-- Developer Console (F9) ⟩ Network
```

Every web call the game has made: `HttpService` requests and the ones Roblox services such as `DataStoreService` make. **Summary** counts calls, failures and time by type; **Details** lists each call's method, status, time, type and URL.

## TextService, PolicyService and the ban API

Text one player writes for others is filtered on the server before anyone sees it. Rule-breakers can be kicked or banned across every server.

### `FilterStringAsync`

```luau
TextService:FilterStringAsync(stringToFilter: string, fromUserId: number, textContext: Enum.TextFilterContext?) -> TextFilterResult  -- yields; default PrivateChat
```

Filters text a player wrote, once per submission. It throws if the author is not in this server, and on a service error it has already retried internally, so do not retry: show nothing.

> `textContext` is `PublicChat` or `PrivateChat`; the docs say it does not change the result. Filter after the player submits, not on every keystroke. Chat through `TextChatService` is filtered for you.

### `GetNonChatStringForBroadcastAsync, GetNonChatStringForUserAsync`

```luau
TextFilterResult:GetNonChatStringForBroadcastAsync() -> string  -- yields
TextFilterResult:GetNonChatStringForUserAsync(toUserId: number) -> string  -- yields
```

The filtered text, either for everyone in the server, such as a sign, or for one reader, filtered by that reader's age and other details. The guide calls these in `pcall` too.

### `Text you store`

```luau
-- save the raw text
-- filter it again each time it is loaded and shown
```

The text filtering guide covers text saved in a DataStore, such as a pet's name: it may contain inappropriate words, so filter it when it is retrieved. Roblox removes an experience it finds is not filtering until filtering is added.

### `GetPolicyInfoForPlayerAsync`

```luau
PolicyService:GetPolicyInfoForPlayerAsync(player: Player) -> {[string]: any}  -- yields
```

What local rules allow for this player, from their location, age group and platform: flags such as `ArePaidRandomItemsRestricted`, `IsPaidItemTradingAllowed`, `AreAdsAllowed` and `IsSubjectToChinaPolicies`. An experience with the matching feature must check the flag.

> Call it in `pcall`. On the client it works only for the local player.

### `BanAsync`

```luau
Players:BanAsync(config: {
  UserIds: {number},          -- at most 50
  Duration: number,           -- seconds; -1 is permanent
  DisplayReason: string,      -- at most 400 characters, filtered
  PrivateReason: string,      -- at most 1,000 characters, never shown
  ApplyToUniverse: boolean?,  -- default true
  ExcludeAltAccounts: boolean?,  -- default false
  ApplyDeviceBlock: boolean?, -- default false
})  -- yields
```

Bans users and evicts them from every server of the place or universe. The banned player sees `DisplayReason` and the time left. By default the ban covers every place and suspected alternate accounts; `ApplyDeviceBlock` also blocks their device from rejoining for 24 hours.

> Server only. It makes one HTTP call per user and joins any failures into one error message, so call it in `pcall`. Bans made in Studio do not apply to the live game.

### `UnbanAsync, GetBanHistoryAsync`

```luau
Players:UnbanAsync(config: { UserIds: {number}, ApplyToUniverse: boolean? })  -- yields
Players:GetBanHistoryAsync(userId: number | User) -> BanHistoryPages  -- yields
```

`UnbanAsync` lifts only bans with the same `ApplyToUniverse` scope, and also lifts a device block. `GetBanHistoryAsync` pages through a user's bans and unbans; it works only on live servers.

### `BanningEnabled`

```luau
Players.BanningEnabled: boolean  -- set in Studio
```

Turns the three ban methods on and off. Scripts cannot set it: switch it on in the Players service's Properties window.

### `Kick`

```luau
Player:Kick(message: string?)
```

Disconnects the player from this server, with your message or a default one. It only disconnects them: `BanAsync` is what keeps a player from rejoining.

> Called from a LocalScript it can kick only the local player.

## HttpService, secrets and ConfigService

Tools for a running game: calling outside services with keys kept out of your code, changing values without a restart, and knowing when servers will restart.

### `RequestAsync`

```luau
HttpService:RequestAsync(requestOptions: {
  Url: string | Secret,
  Method: string?,
  Headers: {[string]: string | Secret}?,
  Body: string?,
}) -> { Success: boolean, StatusCode: number, StatusMessage: string, Headers: {[string]: string}, Body: string }  -- yields
```

Sends an HTTP request from the server. `Success` is true only for status codes 200 to 299. It throws when the request times out or is rejected, so call it in `pcall`. The `GetAsync` and `PostAsync` pages say to use it in most cases.

> HTTP requests are off until **Allow HTTP Requests** is on in Experience Settings ⟩ Security. The docs limit a server to 500 external requests a minute. Set `Content-Type` yourself; `RequestAsync` does not detect it.

### `GetSecret`

```luau
HttpService:GetSecret(key: string) -> Secret
Secret:AddPrefix(prefix: string) -> Secret
Secret:AddSuffix(suffix: string) -> Secret
```

Reads a value from the experience's secrets store, created on the Creator Dashboard. A `Secret` cannot be printed (it prints as `Secret(name)`), but prefixes and suffixes build it into a URL or a header such as `Bearer …`.

> Secrets work on live servers and in collaborative testing. In a local playtest the call errors unless the secret is added under **Local Secrets** in Experience Settings ⟩ Security. A secret can go in the URL or headers, never the body.

### `JSONEncode, JSONDecode`

```luau
HttpService:JSONEncode(input: any) -> string
HttpService:JSONDecode(input: string) -> any
```

Converts between Luau tables and JSON, for request bodies and for measuring how large a DataStore value is. Both are thread-safe.

### `ScriptContext.Error`

```luau
ScriptContext.Error(message: string, stackTrace: string, script: Instance)  -- event
```

Fires when a script thread ends with an unhandled error, with the message, a formatted stack trace and the script. It is the hook for sending errors to your own reporting service.

> It does not fire for the timeout that stops a script running too long.

### `GetConfigAsync, GetValue`

```luau
ConfigService:GetConfigAsync() -> ConfigSnapshot  -- yields, server only
ConfigSnapshot:GetValue(key: string) -> any
```

Experience configs are keys and values edited on the Creator Hub or in Studio and published without a new place version. A snapshot holds every value at the moment it was taken; `GetValue` returns `nil` for an unknown key.

> `GetConfigAsync` throws only in the rare case that configs fail to load and never have before; after a lost connection it returns the last values it had. Keep fallback values in code.

### `UpdateAvailable, Refresh`

```luau
ConfigSnapshot.UpdateAvailable()  -- event
ConfigSnapshot:Refresh()
ConfigSnapshot:GetValueChangedSignal(key: string) -> RBXScriptSignal
```

A snapshot does not change by itself. `UpdateAvailable` fires when newer values are published; call `Refresh` to take them, at a moment that suits the game. `GetValueChangedSignal` listens for a refresh that changes one key.

> `ConfigService:SetTestingValue(key, value)` overrides a value on this server only, for testing, until `ClearTestingValue`.

### `ServerRestartScheduled`

```luau
DataModel.ServerRestartScheduled(restartTime: DateTime, source: Enum.CloseReason, attributes: {[string]: any})  -- event
```

Fires when a restart of this server is scheduled: `source` is `DeveloperUpdate` when you restart servers with a delay to release an update, and `RobloxMaintenance` for maintenance. Use it to warn players and let them save.

> The update guide's **Restart Servers** stops matchmaking to the servers being restarted, waits any delay you chose (1 to 60 minutes), then teleports everyone on an old server together to one server running the new version. Without a restart, old servers run until they empty.

## Limits

Every number the course relies on, as the docs gave it when this course was verified. Limits change, so the linked page wins over this one.

### `DataStore value and storage`

```luau
Value: 4,194,304 characters per key
Storage: (500 + 1 × lifetime users) MB
```

The largest value one key can hold, and the experience's total storage, which grows with every user who has ever joined.

> The value is stored as a string whatever its type; `HttpService:JSONEncode` shows its length. Storage is measured on the compressed latest version of each key; older versions and deleted keys do not count. A lifetime user is anyone who has ever joined. Source: [error-codes-and-limits#data-limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#data-limits), [error-codes-and-limits#storage-limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#storage-limits).

### `DataStore server budgets`

```luau
StandardRead: (60 + 40 × players in this server) requests per minute
StandardWrite: (60 + 40 × players in this server) requests per minute
StandardList: (5 + 2 × players in this server) requests per minute
StandardRemove: (60 + 40 × players in this server) requests per minute
OrderedRead: (60 + 40 × players in this server) requests per minute
OrderedWrite: (30 + 5 × players in this server) requests per minute
OrderedList: (5 + 2 × players in this server) requests per minute
OrderedRemove: (30 + 5 × players in this server) requests per minute
Request queue: 30 requests per queue
```

What one server may spend, by `Enum.DataStoreRequestType`, and how many throttled requests can wait. `UpdateAsync` spends a read and a write. `SetRateLimitForRequestType` can change these; `GetRequestBudgetForRequestType` shows what is left.

> Throttled requests wait in a queue per request kind; once one is full, new requests fail with an error from 301 to 306. Source: [error-codes-and-limits#server-limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#server-limits), [error-codes-and-limits#limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#limits).

### `DataStore experience budgets`

```luau
StandardRead: (300 + 40 × concurrent users) requests per minute
StandardWrite: (300 + 20 × concurrent users) requests per minute
StandardList: (300 + 2 × concurrent users) requests per minute
StandardRemove: (300 + 40 × concurrent users) requests per minute
OrderedRead: (300 + 40 × concurrent users) requests per minute
OrderedWrite: (300 + 20 × concurrent users) requests per minute
OrderedList: (300 + 2 × concurrent users) requests per minute
OrderedRemove: (300 + 40 × concurrent users) requests per minute
```

What every server together may spend. Open Cloud requests draw on the same budget, so a busy external tool can throttle the game and the reverse.

> Source: [error-codes-and-limits#access-limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#access-limits).

### `DataStore per-key throughput`

```luau
Read: 25 MB per minute per key
Write: 4 MB per minute per key
```

How much data one key can move, across every server. Past it, requests fail with `KeyThrottled`.

> Throughput counts the last 60 seconds across every server, rounding each request up to the next kilobyte. Source: [error-codes-and-limits#throughput-limits](https://create.roblox.com/docs/cloud-services/data-stores/error-codes-and-limits#throughput-limits).

### `GetAsync cache`

```luau
GetAsync cache: 4 seconds
```

How long a read is served from this server's local cache.

> A cached read counts against neither the server budget nor per-key throughput. Source: [versioning-listing-and-caching#caching](https://create.roblox.com/docs/cloud-services/data-stores/versioning-listing-and-caching#caching).

### `DataStore pages and versions`

```luau
GetSortedAsync page: 100 items per page
Version backups: 30 days after a newer write replaces one
```

The largest page `GetSortedAsync` returns, and how long an overwritten version stays readable.

> A page holds 50 items unless `pageSize` asks for another number. `ListKeysAsync` also defaults to 50 keys a page, but its page gives no maximum. The latest version never expires. Source: [OrderedDataStore#GetSortedAsync](https://create.roblox.com/docs/reference/engine/classes/OrderedDataStore#GetSortedAsync), [versioning-listing-and-caching#versioning](https://create.roblox.com/docs/cloud-services/data-stores/versioning-listing-and-caching#versioning).

### `MessagingService`

```luau
Message: 1 kB
Topic: 80 characters
Sent: (600 + 240 × players in this server) messages per minute per server
Subscriptions: (20 + 8 × players in this server) subscriptions per server
Received per topic: (40 + 80 × servers) messages per minute
Received in all: (400 + 200 × servers) messages per minute
```

Per message and topic name, what one server may send and subscribe to, and what can be received per topic and across every server.

> A topic needs at least 1 character. The MessagingService page says its limits are subject to change. Source: [MessagingService](https://create.roblox.com/docs/reference/engine/classes/MessagingService).

### `MemoryStore`

```luau
Memory: (64 + 1.2 × concurrent users) KB
Request units: (1,000 + 120 × concurrent users) request units per minute
Expiration: 3,888,000 seconds (45 days)
Items: 1,000,000 per sorted map or queue
Size: 100 MB per sorted map or queue
```

Quotas for the whole experience, the longest an item can live, and the size of one sorted map or queue.

> The memory quota is game-wide. It grows as soon as users join but takes eight days to shrink after they leave, and once it is full any request that would add memory fails. The request unit quota is game-wide too. Most calls cost one unit; `GetRangeAsync` and `ReadAsync` cost one per item returned, and hash map `UpdateAsync` at least two. A sorted map's size includes its keys. Source: [memory-stores#memory-size-quota](https://create.roblox.com/docs/cloud-services/memory-stores#memory-size-quota), [memory-stores#api-request-limits](https://create.roblox.com/docs/cloud-services/memory-stores#api-request-limits), [MemoryStoreSortedMap#SetAsync](https://create.roblox.com/docs/reference/engine/classes/MemoryStoreSortedMap#SetAsync), [memory-stores#data-structure-size-limits](https://create.roblox.com/docs/cloud-services/memory-stores#data-structure-size-limits).

### `MemoryStore reads`

```luau
GetRangeAsync: 200 items per call
ReadAsync: 100 items per call
```

The most items one `GetRangeAsync` or `ReadAsync` call returns.

> Source: [MemoryStoreSortedMap#GetRangeAsync](https://create.roblox.com/docs/reference/engine/classes/MemoryStoreSortedMap#GetRangeAsync), [MemoryStoreQueue#ReadAsync](https://create.roblox.com/docs/reference/engine/classes/MemoryStoreQueue#ReadAsync).

### `Badges`

```luau
AwardBadgeAsync: (50 + 35 × users) requests per minute
```

How often a game may award badges.

> The BadgeService page does not say whether it counts the users in one server or in the whole experience. Source: [BadgeService#AwardBadgeAsync](https://create.roblox.com/docs/reference/engine/classes/BadgeService#AwardBadgeAsync).

### `Bans`

```luau
UserIds: 50 users per call
DisplayReason: 400 characters
PrivateReason: 1,000 characters
Device block: 24 hours
```

What one `BanAsync` call can carry, and how long a device block lasts.

> `DisplayReason` is text filtered; `PrivateReason` is not, and never reaches the client. Of the ways to unban a user, only `UnbanAsync` also lifts the device block. Source: [Players#BanAsync](https://create.roblox.com/docs/reference/engine/classes/Players#BanAsync).

### `Shutdown`

```luau
BindToClose: 30 seconds
```

How long the server waits for `BindToClose` functions.

> After the window the server shuts down even if bound functions are still running. Source: [DataModel#BindToClose](https://create.roblox.com/docs/reference/engine/classes/DataModel#BindToClose).

### `Server restarts`

```luau
Restart delay: 60 minutes at most
```

The longest delay **Restart Servers** gives players before their servers shut down.

> A delay is at least 1 minute. Source: [update-games#restart-servers](https://create.roblox.com/docs/projects/update-games#restart-servers).

### `Teleports`

```luau
TeleportAsync: 50 players per call
Join data: 48 hours
```

How many players one `TeleportAsync` call can move, and how recently teleport data must have been sent for `GetJoinData` to return it. The docs publish no size limit for teleport data.

> `GetJoinData` returns teleport data only if a Roblox server sent it within this time. Source: [TeleportService#TeleportAsync](https://create.roblox.com/docs/reference/engine/classes/TeleportService#TeleportAsync), [Player#GetJoinData](https://create.roblox.com/docs/reference/engine/classes/Player#GetJoinData).

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Scribe](https://scribe.ericplane.dev/) | Player data library with session locking, receipts, migrations and leaderboards, and Kalo's course on it | 2.3.0 | `Scribe = "ericplane/scribe@2.3.0"` |
| [Blink](https://github.com/1Axen/blink) | Networking IDL: typed, compact remotes generated from one schema file | 0.18.8 | `rokit add 1Axen/blink` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **The "berezaa method", DataStore2's default `OrderedBackups` saving.** Every save goes to a new key and an OrderedDataStore tracks the newest, so old saves survive; DataStore2's own docs call it extremely redundant. DataStores now keep versions themselves, and every one of those extra keys counts against the storage quota. Save to one key per player and roll back with `ListVersionsAsync` and `GetVersionAtTimeAsync`.
- **Saving with `SetAsync` alone.** `SetAsync` does not read first, so a server holding stale data overwrites a newer save. The docs call `UpdateAsync` the safer choice when several servers can write a key; pair it with a session lock.
- **`store:GetAsync(key)` without `pcall`.** Every DataStore call is a web request that can fail or be throttled, and an unprotected error ends the script, often mid-load. Wrap each call, and never treat a failed load as an empty profile, or the next save wipes the player's data.
- **Saving only on `PlayerRemoving`.** A server that shuts down can close before those saves finish. The `BindToClose` docs say any game using DataStores should also bind a function there that saves all unsaved data; wait in it for saves already in flight. The deprecated `game.OnClose` callback is the old form of the same hook.
- **`TeleportToPrivateServer`, `TeleportToPlaceInstance`, `TeleportPartyAsync` and `TeleportToSpawnByName`.** All deprecated for `TeleportService:TeleportAsync(placeId, players, options)`, with `ReservedServerAccessCode`, `ServerInstanceId` or `ShouldReserveServer` set on a `TeleportOptions`. `ReserveServer` is deprecated too: use `ReserveServerAsync`.
- **`TeleportService:Teleport(placeId)` from a LocalScript.** The dump does not tag it, but the docs say it is deprecated and should not be used for new work. Fire a RemoteEvent and call `TeleportAsync` on the server, which then lets the experience require server-initiated teleports.
- **`MarketplaceService:PlayerOwnsAsset` for game passes.** Passes have their own IDs and their own check, `UserOwnsGamePassAsync(userId, passId)`. `PlayerOwnsAsset` itself is deprecated for `PlayerOwnsAssetAsync`.
- **`BadgeService:AwardBadge`, `UserHasBadge` and `MarketplaceService:GetProductInfo`.** Deprecated. Write `AwardBadgeAsync`, `UserHasBadgeAsync` and `GetProductInfoAsync`, which take the same arguments.
- **`MarketplaceService.ProcessReceipt = function(receiptInfo) … end`.** The docs now call it the legacy callback. Bind a handler per product with `MarketplaceService:BindReceiptHandler(Enum.ReceiptType.DeveloperProduct, handler, { productId })`, returning `Enum.ReceiptDecision.Processed` or `NotProcessedYet` instead of `Enum.ProductPurchaseDecision`. Receipts no bound handler matches still reach `ProcessReceipt`.
- **A receipt handler that grants and returns before saving, or returns nothing.** The receipt is complete as soon as the handler says so, so a save that then fails loses the purchase, and a handler that does not check `PurchaseId` grants twice. Grant, record the `PurchaseId` and save, then return `Processed`; on any failure return `NotProcessedYet`. Never grant from `PromptProductPurchaseFinished`.
- **Autosave loops written with `spawn` and `wait`.** Both globals are deprecated. Write `task.spawn(function() while true do task.wait(60) … end end)`, or `task.delay`, and stop the loop when the server shuts down.
- **Teleport data used as trusted state.** Teleport data passes through the client, and the docs warn it can be abused even though `GetJoinData` checks where it came from. Send an ID, keep the real data in a DataStore or MemoryStore, and look it up on arrival.
- **`GetRequestBudgetForRequestType(Enum.DataStoreRequestType.GetAsync)` and the other old request types.** The enum reference says `GetAsync`, `SetIncrementAsync`, `UpdateAsync`, `GetSortedAsync`, `SetIncrementSortedAsync`, `OnUpdate`, `ListAsync`, `GetVersionAsync` and `RemoveVersionAsync` should not be used. Pass `StandardRead`, `StandardWrite`, `StandardList`, `StandardRemove` or their `Ordered` equivalents.
- **`store:OnUpdate(key, callback)`.** Deprecated: it polled other servers' changes once a minute. The docs say to publish changes with `MessagingService` instead.
- **DataStore scopes (`GetDataStore("Inventory", "gold")`).** The docs call scopes a legacy feature: new experiences should use key prefixes and list them with `ListKeysAsync(prefix)`. Experiences that already use scopes should keep them.
- **`TextFilterResult:GetChatForUserAsync`.** Deprecated, and it now returns an empty string. Chat is filtered by `TextChatService`; other text uses `GetNonChatStringForBroadcastAsync` or `GetNonChatStringForUserAsync`.
