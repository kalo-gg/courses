# Scribe reference

*Scribe 2.3.0, released September 4, 2026. Checked on September 13, 2026.*

---

## Setup and configuration

One call builds both halves. Three options are required; everything else has a default.

### `Scribe(options)`

```luau
Scribe(options: ScribeOptions<T>) -> { Server: ServerData<T>, Client: ClientData<T> }
Scribe.new(options)  -- the same call
```

Compiles the template, wires the realm you are in, and returns the bundle. Called from a server Script it builds the server half; from a LocalScript, the client half. Both halves are on the returned table, and the one that does not belong to the current realm errors when touched.

> Call it once per realm and share the result. A second call with the same ProfileStoreIndex in one server is refused as a duplicate.

### `{ Server, Client }`

```luau
local Data = Scribe(options).Server   -- ServerScriptService
local Data = Scribe(options).Client   -- StarterPlayerScripts
```

The bundle. `Server` holds every player's tree (`Data[player]`), the lifecycle hooks and the persistence API; `Client` is one player's mirror with root accessors on the object itself (`Data.Coins.Get()`).

### `ProfileStoreIndex, ProfileKeyPrefix`

```luau
ProfileStoreIndex: string  -- the DataStore name
ProfileKeyPrefix: string   -- key = prefix .. userId
```

Required with `Template`. Together they name the DataStore and the key each profile lives under. Change either and every player starts from a fresh profile.

### `Mode`

```luau
Mode: "Live" | "Mock" | "NoSave"
TargetUserId: number?   -- NoSave: load this user's profile instead
```

Which store the bundle talks to. `Live` is real profiles and boards; `Mock` an in-memory store for both, profiles loading as template defaults; `NoSave` loads the real profile as a snapshot and never saves. `ResetData = true` wipes profiles to defaults.

> The legacy flags `UseMock`, `DontSave`, `ViewedUserId` and `OverriddenUserId` map onto Mode and lose to it when both are set. The sandbox forces `Mock`.

### `SaveInterval, Scribe.Configure`

```luau
SaveInterval: number?  -- seconds between autosaves, default 300, floor 15
Scribe.Configure({ AutoSaveInterval: number? })
```

How often each loaded profile is saved when dirty. `Scribe.Configure` sets the process-wide autosave interval before the first bundle is built, so several bundles share it.

### `Scribe.Version, Banner`

```luau
Scribe.Version: string  -- "2.3.0"
Banner: boolean?         -- default true
```

The version string, and whether the startup line is printed. Every lesson sets `Banner = false` to keep the output to what the code prints.

### `The type exports`

```luau
Scribe.PlayerData<T>    -- one player's accessor tree
Scribe.ServerData<T>    -- the server half
Scribe.ClientData<T>    -- the client half
Scribe.ScribeOptions<T> -- the options table
```

`T` is `typeof(template)`. Name a service's parameter `Scribe.PlayerData<typeof(template)>` and the tree is typed end to end. When a large template makes the `Scribe({ … })` call too complex to typecheck, annotate the options as `ScribeOptions<T>`, cast the call and annotate the half you use.

> The typed tree needs the new Luau type solver. Scribe runs the same without it; `Data.Raw` is the untyped escape hatch.

## Declarators

A template is a table of declarators. Each one names a default and the rules every write is checked against.

### `Scribe.Int`

```luau
Scribe.Int(default: number, opts?: { Min?: number, Max?: number })
```

An integer field. Writes outside the bounds are clamped, and every write returns the value that was stored, so a clamp is visible at the call site.

> `Precision` belongs to `Number`; on an `Int` it is refused, not ignored.

### `Scribe.Number`

```luau
Scribe.Number(default: number, opts?: { Min?: number, Max?: number, Precision?: number })
```

A float field with optional bounds and a rounding precision (`Precision = 0.01` stores cents).

### `Scribe.String`

```luau
Scribe.String(default: string, opts?: { MaxLength?: number })
```

A string field. A write longer than `MaxLength` is refused with an error naming the field.

### `Scribe.Enum`

```luau
Scribe.Enum(default: string, values: { string })
```

One of a fixed list of strings. Used as a command `Args` spec it refuses anything off the list before the handler runs.

### `Scribe.Optional`

```luau
Scribe.Optional(inner: Declarator)
```

A field with no default: `nil` until written, then checked by the inner declarator. Inside a container shape it is the way to say an entry may lack the key.

### `Scribe.Dynamic`

```luau
Scribe.Dynamic(inner: Declarator, compute: (player: Player) -> any)
```

A per-profile default computed once, when the profile is first created. Replaces the most common reason to write an `OnPlayerInit`.

### `Scribe.Big`

```luau
Scribe.Big(default: number | BigValue, opts?: { Min?: number })
```

A number that grows past 2^53 without losing precision, for idle-game currencies. Reads return a `BigValue`; writes accept numbers or BigValues.

### `Scribe.Derived`

```luau
Scribe.Derived(shape: Declarator, inputs: { string }, compute: (...any) -> any)
```

A field computed from other fields. It is never stored, recomputes when any input changes, and cannot be written. A derived field whose inputs persist can back a leaderboard.

> A derived function that throws is reported as `DERIVED_ERROR`. Derived is refused under a persisted record inside a container.

### `Scribe.Timed`

```luau
Scribe.Timed(default: any)
```

A value with an expiry. `SetTimed(value, seconds)` arms it; when the deadline passes a once-per-second sweep writes the default back and `Changed` fires. `Active()` answers `(active, remaining)`.

> The sweep is jittered, so a lapse lands up to about a second after the deadline. Refused inside a container's element shape.

### `The datatypes`

```luau
Scribe.Vector3(default), Scribe.CFrame(default, { Precision = "exact"? }), Scribe.Color3(default), …
Scribe.Datatypes.Pack(value) / Unpack(packed)
```

Roblox datatypes stored as packed tables: Vector3, Vector2, CFrame, Color3, UDim2, BrickColor and their siblings. `Pack` and `Unpack` are what you use when writing one into the raw table in `OnPlayerInit` or a migration.

## Reading and writing

Every field in the tree is an accessor with the same small set of methods.

### `Get`

```luau
field.Get() -> T
```

The current value. A container's `Get()` returns a copy of the stored table, so edits to it change nothing; write through the accessor instead.

### `Set`

```luau
field.Set(value: T, replicate?: boolean) -> T
```

Stores a value after checking it against the declarator, and returns what was stored. The second argument overrides replication for this write: `false` keeps a plain field's write off the wire.

### `Update`

```luau
field.Update(fn: (current: T) -> T, replicate?: boolean) -> T
```

Reads, computes, writes, as one call: the safe way to change a value that depends on its current value.

### `Clone`

```luau
field.Clone() -> T
```

A deep copy of the current value, for a table you intend to mutate before writing back.

### `Default, Min, Max`

```luau
field.Default() -> T
field.Min() -> number?
field.Max() -> number?
```

The declarator's answers, readable at runtime: what the field starts as, and the bounds a write is clamped to.

### `Increment, Decrement`

```luau
field.Increment(delta: number, tags?: EconomyTags | false) -> number
field.Decrement(delta: number, tags?: EconomyTags | false) -> number
```

Add or subtract on a number field, clamped to the bounds, returning the stored value. The second argument carries economy tags (`{ Source = "quest" }`) for the analytics log, or `false` to skip logging.

### `Toggle`

```luau
field.Toggle() -> boolean
```

Flips a boolean field and returns the new value.

### `Observe, Changed`

```luau
field.Observe(fn: (value: T) -> ()) -> () -> ()
field.Changed(fn: (new: T, old: T) -> ()) -> () -> ()
```

`Observe` calls `fn` now and after every change; `Changed` only after a change, with both values. Both return a disconnect function. On the client they are how a HUD follows the mirror.

> A container's `Changed` no longer receives a third `key` argument; use `OnChildChanged` for that.

## Containers

Typed tables: every entry is checked against one shape, and each kind adds the methods its shape needs.

### `Scribe.DictOf`

```luau
Scribe.DictOf(shape: table, opts?: { MaxKeys?: number })
data.Inventory[key].Set(entry) / .Get()
data.Inventory.Count() / Remove(key) / Has(key) / Clear()
```

A dictionary of records under string keys. An unresolved key reads `nil`; writing through it creates the entry, filling omitted fields from their defaults. `MaxKeys` refuses the entry past the cap.

### `Scribe.MapOf`

```luau
Scribe.MapOf(keyDeclarator: Declarator, valueDeclarator: Declarator, opts?)
```

A dictionary whose keys are checked too, for maps keyed by numbers or enums rather than free strings.

### `Scribe.ArrayOf`

```luau
Scribe.ArrayOf(shape: Declarator | table, opts?: { MaxItems?: number, Evict?: "Front" | "Back" })
arr.Insert(value) / Remove(index) / RemoveValue(value) / Find(value) / Count() / Clear()
```

An ordered list with a cap. With `Evict = "Front"` an insert past `MaxItems` drops the oldest entry, which is how a last-five-runs window works; without it the insert is refused.

### `Scribe.SetOf`

```luau
Scribe.SetOf(shape: Declarator, opts?: { MaxItems?: number })
set.Add(value) -> boolean  /  Has(value) / Remove(value) / Count() / Clear()
```

Unordered, no duplicates. `Add` returns whether the value was new, which makes a first-time bonus one `if`.

### `Scribe.Flags`

```luau
Scribe.Flags(names: { string })
flags.Enable(name) / Disable(name) / Toggle(name) / Has(name) / Clear()
```

A fixed set of named booleans stored as the list of enabled names. In the raw table (hooks, migrations) a Flags field is that list.

### `Big arithmetic and BigValue`

```luau
big.Multiply(factor, replicate?) / big.Divide(factor, replicate?)
BigValue.Short(digits?) / ToNumber() / Pow(n) / Log10()  ·  .M, .E
Scribe.Short(n) / Scribe.SetShortSuffixes(list)
```

A `Big` field adds multiply and divide; dividing by zero errors naming the field. A `BigValue` is mantissa and exponent, formats itself with `Short()` (`1.2M`), and converts with `ToNumber()` when it fits. `Scribe.Short` formats plain numbers the same way.

### `The container events`

```luau
container.OnChildChanged(fn: (key, new, old) -> ())
dict.OnKeyAdded(fn: (key, entry) -> ()) / OnKeyRemoved(fn: (key, entry) -> ())
arr.OnInsert(fn: (element, index) -> ()) / OnRemove(fn: (element, index) -> ())
```

Structural listeners on containers, each returning a disconnect function. `Changed` on the container still fires with the whole value; these say what moved. Writes inside one `Batch` coalesce to one fire per listener.

## Replication, the client and commands

The server owns the tree; the client holds a mirror and asks for changes through commands.

### `ServerOnly, Shared, Session`

```luau
Scribe.ServerOnly(shape)  -- never leaves the server
Scribe.Shared(shape)      -- every client may read it
Scribe.Session(shape)     -- never saved
```

Wrappers around a field or group. Plain fields replicate to their owner only; `Shared` replicates to everyone; `ServerOnly` to nobody; `Session` lives for the session and is dropped from the save. A `Session` field cannot back a leaderboard.

### `The client object`

```luau
Client.IsReady() -> boolean
Client.WaitForData(timeout?: number) -> boolean
Client.Coins.Get() / .Observe(fn)   -- root accessors on the object
Client.Mock(fields, extras?) / Client.MockCommand(name, fn)  -- edit mode only
```

The mirror. It handshakes with the server on build and fills in on the first diff; `WaitForData` answers whether that happened in time. In edit mode (`RunService:IsRunning()` false) it initialises instantly to defaults, and `Mock` and `MockCommand` seed it.

> Client-side `Set` only touches the mirror. There is no write frame; the next diff overwrites it.

### `GetShared, OnSharedChanged`

```luau
Client.GetShared(playerOrUserId) -> table?
Client.OnSharedChanged:Connect(fn: (userId: number, shared: table?) -> ())
```

Other players' `Shared` groups, as plain tables. The signal fires with the whole shared table when any of it changes, and with `nil` when that player leaves.

### `Data.Command`

```luau
Data.Command(name: string, spec?: { Args?: { Declarator | string }, Idempotent?: boolean }, handler: (player, ...args) -> (boolean, ...any))
CommandRateLimit: number?  -- default 20 per second per player
RequestTimeout: number?
```

Registers a server handler a client can request by name. `Args` declarators (or type-name strings) are checked before the handler runs; a mismatch answers `bad-args`. `Idempotent = true` lets `RequestOnce` replay a reply under its key.

> The player argument is the only identity to trust; a caller-supplied user id is only ever acceptable after that identity passes a permission check.

### `Client.Request`

```luau
Client.Request(name: string, ...args) -> (ok: boolean, reason: string?, failure: RequestFailed?)
```

Sends a command and yields for the reply. `ok` and the handler's second value come back as the handler returned them; a refusal by Scribe itself carries a `Scribe.RequestReason` string and a `RequestFailed` table with detail.

### `Client.RequestOnce`

```luau
Client.RequestOnce(name: string, key: string, ...args) -> (ok: boolean, reason: string?)
```

`Request` for an idempotent command. The first reply under `key` is stored on the profile, and a repeat with the same key returns it without running the handler again: a double click is free.

### `Scribe.RequestFailed, Scribe.RequestReason`

```luau
Scribe.RequestReason.BadArgs == "bad-args"  ·  RateLimited  ·  NotReady  ·  UnknownCommand  ·  Timeout  ·  Error  ·  …
RequestFailed = { Reason, Command, Detail? }
```

The named refusals Scribe answers with when a request never reached, or never returned from, your handler, and the table describing one. Compare against the constants, never the strings.

## Sessions, saving and time

From PlayerAdded to the last save: waiting, hooks, flushes, transactions and timers.

### `WaitForData, Scribe.Reason`

```luau
Data.WaitForData(player: Player, timeout?: number) -> (data: PlayerData?, reason: string?)
Scribe.Reason.StillLoading  ·  PlayerLeft  ·  Timeout  ·  LoadFailed  ·  MigrationFailed  ·  SessionEnded  ·  Shutdown
```

Yields until the player's profile is Ready, up to 60 seconds by default, and returns the accessor or one of seven reasons. Only `StillLoading` is worth a retry: the wait elapsed while the load was still in flight.

### `Data[player], Data.Get, Data.GetState`

```luau
Data[player] -> PlayerData          -- errors if not Ready
Data.Get(player) -> PlayerData?
Data.GetState(player) -> "Loading" | "Ready" | "SessionEnded"
```

Non-yielding access to a loaded player's tree. Index the bundle when you know the player is Ready; `Get` answers `nil` otherwise; `GetState` says which.

### `OnPlayerInit, OnPlayerLeaving`

```luau
OnPlayerInit: (player, rawData: table, isNewProfile: boolean) -> ()
OnPlayerLeaving: (player, data: PlayerData, reason: string) -> ()
```

Two options. `OnPlayerInit` runs after the load and before Ready, against the raw table (writes bypass validation; a Flags field is its enabled names; derived fields are absent). `OnPlayerLeaving` runs with the accessor before the final save, must not yield, and is skipped for a stolen session.

### `OnSave, SessionEnded, GetSaveInfo`

```luau
Data.OnSave:Connect(fn: (info: { Player, Ok, Duration, At }) -> ())
Data.SessionEnded:Connect(fn: (player, reason) -> ())
Data.GetSaveInfo(player) -> { Dirty: boolean, LastSaveAt: number?, … }
```

Signals (`:Connect`, `:Once`, `:Wait`) for every save and every session end, and a per-player snapshot of the save state.

> `Data.OnSave` is a Signal, not a function to call: `Data.OnSave:Connect(fn)`.

### `Flush, Stop`

```luau
Data.Flush(player: Player) -> boolean
Data.Stop()
```

`Flush` saves now and answers whether the bytes are confirmed; on a clean profile it answers true immediately without a request. A false is unresolved, never a reason to compensate. `Stop` releases the bundle; players must have left first.

### `Batch, Transaction`

```luau
Data.Batch(player, fn: () -> ()) -> ()
Data.Transaction(player, fn: () -> ()) -> (ok: boolean, err: string?)
```

`Batch` coalesces listeners and replication for a group of writes. `Transaction` also rolls every write back if `fn` errors, yields, or touches another player's tree; `error("reason", 0)` inside it is how a handler refuses. Neither saves: `Flush` is the commit point.

### `Timed fields and cooldowns`

```luau
timed.SetTimed(value, seconds) / timed.Active() -> (active: boolean, remaining: number) / timed.ExtendTimed(seconds)
Data.OnCooldown(player, key: string, seconds: number) -> (onCooldown: boolean, remaining: number)
Data.PeekCooldown(player, key) / ClearCooldown(player, key) / OnCooldownEnded:Connect(fn)
```

A timed field holds a value the game reads and expires it; a cooldown is a gate nothing reads. `OnCooldown` both checks and arms: true means refuse, false means the action happened and the cooldown is now running. Cooldowns persist in the profile and never replicate.

## Offline profiles, purchases and leaderboards

Acting on profiles nobody is playing, money in both directions, and rankings from one line of config.

### `GetOffline, UpdateOffline`

```luau
Data.GetOffline(userId: number) -> table?
Data.UpdateOffline(userId: number, fn: (raw: table) -> ()) -> (ok: boolean, reason: string?)
```

A raw snapshot with derived fields filled in, and an edit committed by compare-and-set. Every offline call yields. `UpdateOffline` fails closed while the user has a live session on any server, and answers false without writing if the profile changed under it.

### `ListVersions, GetVersion, RestoreVersion`

```luau
Data.ListVersions(userId, limit?) -> { { Id, At } }
Data.GetVersion(userId, versionId) -> table?
Data.RestoreVersion(userId, versionId, opts?: { RollBackReserved?: boolean }) -> (ok, reason?)
```

Version history, live DataStore only (Mock mode has none). A restore keeps the reserved `_Scribe` root (receipts, perks, cooldowns) from the live profile unless `RollBackReserved = true`, and fails closed over a live session.

### `Export, Erase`

```luau
Data.Export(userId) -> string   -- JSON, buffers base64-encoded
Data.Erase(userId) -> (ok: boolean, reason: string?)
```

The GDPR pair. `Export` is the profile as JSON; `Erase` deletes the profile and then the user's leaderboard entries, is idempotent, and fails closed over a live session.

### `Migrations`

```luau
Migrations: { [2]: (raw: table) -> (), [3]: …, … }  -- contiguous from 2
VersionAheadPolicy: "Kick" | …  ·  VersionAheadMessage: string?  ·  MigrationShadow: boolean?
```

Numbered steps run during the load, between reconcile and `OnPlayerInit`, against the raw table. A throwing step ends the session with `migration-failed` and saves nothing; a gap in the chain fails startup. Key each step off a field you removed, never off a missing template key, because reconcile backfills those first.

> A step's edits to `_Scribe` are discarded and logged as `MIGRATION_RESERVED_DISCARDED`.

### `SendMessage, OnMessage`

```luau
Data.SendMessage(userId: number, payload: any) -> boolean
Data.OnMessage:Connect(fn: (player, message) -> ())
```

A durable outbox to another player, delivered at least once when they are next loaded. The recipient applies it inside a transaction that also writes a marker, so a redelivery does nothing. The one-sided answer to most cross-player features.

### `Products, Passes, Perks; PromptPurchase, TryHandleReceipt`

```luau
Products: { [name]: { Id: number, Category: string, Grant?: (data) -> () } }
Passes: { [name]: { Id: number } }  ·  Perks: { string }
Data.PromptPurchase(player, name) -> (ok, reason?)
Data.TryHandleReceipt(receipt) -> Enum.ProductPurchaseDecision
```

Named in the options. With any product declared Scribe owns `ProcessReceipt`: it grants, logs and answers `PurchaseGranted` only after the save confirms, otherwise `NotProcessedYet` so Roblox retries. `TryHandleReceipt` is the same path for a game that keeps its own handler. A `Grant` must not yield.

> The sandbox has no purchase prompt, so `PromptPurchase` answers false there; the receipt path runs.

### `Ownership`

```luau
Data.Owns(player, key) -> boolean          -- cached, never yields
Data.OwnsAsync(player, key) -> boolean     -- verifies when the cache says no
Data.GrantPerk(player, key) / RevokePerk(player, key)
Data.OnOwnershipChanged:Connect(fn: (player, key, owned) -> ())
Data.GetProductState(player, key) -> "owned" | "purchasable" | "not-loaded" | …
```

Perks, passes and Premium (`"RobloxPlus"`) answer to one key. Gate a grant on the server's check, never the client's mirror; `Owns` can read false for a moment after join while the refresh runs.

### `Purchase, Scribe.PurchaseReason, GetPurchases`

```luau
Data.Purchase(player, { Cost: { Path, Amount }, Category, ItemId, IdempotencyKey?, Grant?: (data) -> () }) -> (ok, reason?)
Scribe.PurchaseReason.InsufficientFunds  ·  DataNotLoaded  ·  InvalidCostSpec  ·  …
Data.GetPurchases(player, { Kind?: "InGame" | "Robux", Limit? }) -> { entry }
```

A soft-currency purchase: debit, grant and log as one write, all or nothing. A repeat under the same `IdempotencyKey` returns the first result and spends nothing. A `Grant` that yields is refused, because here the debit can be rolled back.

> A grant through an unresolved container key creates the entry silently; resolve the id before you spend on it.

### `Leaderboards`

```luau
Leaderboards: { [name]: { Stat: string, Limit?: number, Replicate?: boolean, RefreshInterval?: number, Scale?: number, SigFigs?: number, StoreName?: string } }
Data.GetLeaderboard(name, n?) -> { { Rank, UserId, Name, Score } }
Data.GetMyRank(player, name) -> number?
Data.OnLeaderboard:Connect(fn: (name, entries) -> ())  -- also on the client with Replicate
```

An all-time ranking of one `Int` or `Big` field on a persisted path. Reads come from a cache the refresh loop fills, so a board is empty until its first refresh (about five seconds after boot) and `GetMyRank` is a rank within the cached top or `nil`. `RefreshInterval` can only go slower than 60 seconds.

## Diagnostics

Stable codes, a health verdict and counters, all on the `Scribe` table itself.

### `OnIssue, OnStatusChanged, GetStatus`

```luau
Scribe.OnIssue:Connect(fn: (entry: LogEntry) -> ())        -- Error and Fatal
Scribe.OnStatusChanged:Connect(fn: (status, previous) -> ())
Scribe.GetStatus() -> "Healthy" | "Degraded" | "Outage"
```

Every failure carries a stable code. `Degraded` changes nothing inside Scribe; only `Outage` refuses, and only receipts, gifts, restores and erases.

### `GetRecentLogs, AddLogSink, LogRingSize, LogLevel, DevMode`

```luau
Scribe.GetRecentLogs(filter?: { Code?, Level?, Category?, Limit? }) -> { LogEntry }
Scribe.AddLogSink(fn: (entry: LogEntry) -> ()) -> detach: () -> ()
LogRingSize: number?  -- default 512
LogLevel: "Debug" | "Info" | "Warn" | "Error"  -- console only
DevMode: boolean?
```

Every entry is `{ At, Level, Category, Code, Message, Context }` and lands in the ring whatever its level; `LogLevel` filters only the console (`Warn` on a live server, `Debug` in Studio). A sink sees entries as they happen; the ring is read oldest-first.

### `GetMetrics, GetPercentiles`

```luau
Scribe.GetMetrics() -> { [name]: number | { Count, Average, Max } }
Scribe.GetPercentiles() -> { [name]: { P50, P90, P99 } }
```

A flat, sparse snapshot: a key exists once something emitted it, so a missing key is zero. Distributions (`SaveDuration`, `ProfileSize`) carry `{ Count, Average, Max }`; the percentiles cover the last 256 samples. `DataStoreErrors_<Class>` splits store errors by class.

### `OnAnomaly and the policies`

```luau
Data.OnAnomaly:Connect(fn: (info: { Player, Path, Value, Reason }) -> ())
BoundsPolicy: "Clamp" | "Reject"  ·  WipeGuardPolicy: "Warn" | "Block"  ·  SchemaPolicy: "Warn" | "Reject"
```

`ANOMALY` fires for a server-side write the template refused: a bug report to yourself, never grounds for player-facing enforcement. The policies decide what a refused write, a save that looks like loss, or an unknown root key does.

### `ServerStore`

```luau
ServerStore: { … }?  -- declared in the types and the docs
```

A per-server (not per-player) store described in the docs and typed in the options. In 2.3.0 `Scribe({ … })` does not pass it to the compiler and reports it as `UNKNOWN_OPTION`, so it is reachable only through the internal `Server.build`. The course does not teach it.

> Upstream defect in 2.3.0; check the changelog of a later release before relying on it.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Scribe](https://scribe.ericplane.dev/) | The library itself | 2.3.0 | `Scribe = "ericplane/scribe@2.3.0"` |
| [ProfileStore](https://github.com/MadStudioRoblox/ProfileStore) | The session-locked DataStore layer underneath | 1.0.3 | `vendored inside the package` |
| [Wally](https://wally.run) | Package manager for Roblox | 0.3.2 | `wally install` |
| [Scribe Studio](https://create.roblox.com/store/asset/113609038046646) | The Studio plugin that renders diagnostics live | Creator Store | `asset 113609038046646` |
| [The UI adapters](https://github.com/ericplane/scribe) | useScribe for Fusion, React and Vide | repository only | `adapters/Fusion.luau, React.luau, Vide.luau` |
| [Lune](https://lune-org.github.io/docs) | How the library tests itself headless | 0.10.5 | `lune run lune/run-tests` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`UseMock = true`, `DontSave`, `ViewedUserId`, `OverriddenUserId`.** The legacy flags. 2.3.0 has `Mode = "Mock" | "NoSave" | "Live"` and `TargetUserId`, and the flags lose to `Mode` when both are set.
- **A container `Changed` listener with a third `key` argument.** Removed in 1.3.0; `Changed` fires with the whole value, and `OnChildChanged(fn(key, new, old))` is the per-key listener.
- **`PromptPurchase` refusing with `player data not loaded` or `player already owns "VIP"`.** 2.3.0 answers with the `Scribe.ProductState` strings, `not-loaded` and `owned`; compare against the table, not the prose.
- **`Data.OnSave(fn)` as a call.** `OnSave` is a Signal: `Data.OnSave:Connect(fn)`. The same goes for every `On…` member on `Data`, `Client` and `Scribe`.
- **`profile.Data.Coins += 50`, ProfileStore style.** Writes go through the accessor, `data.Coins.Increment(50)`, and are checked against the template. The raw table is only ever handed to `OnPlayerInit`, a migration step and `UpdateOffline`.
- **`Scribe.Timed` inside a container's element shape, or `Scribe.Derived` under a persisted record.** Both are refused since the container rework; a timed value or a derived field lives at the root or in a plain group.
- **`ServerStore` in the options.** Documented and typed, but `Scribe({ … })` drops it with `UNKNOWN_OPTION` in 2.3.0. See the Diagnostics group's note.
- **`Scribe.Int(0, { Precision = 0.01 })`.** `Precision` belongs to `Scribe.Number`; on an `Int` it is refused rather than ignored, so the template fails to compile.
