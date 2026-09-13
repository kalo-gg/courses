# Testing and edit mode

*Mode closes off every store a play-test could touch; edit mode renders a storybook with no server anywhere.*

Scribe · Shipping Emberfall · 10 min

---

A play-test that saves can overwrite your own profile or write a junk score onto a live board. `Mode` closes that off: `"Live"` is the real store; `"Mock"` a full in-memory store for both, every profile loading as template defaults; `"NoSave"` loads the **real** profile as a snapshot and never saves, with `TargetUserId = 101` to inspect one player. `ResetData = true` wipes profiles to defaults and is destructive. The legacy flags (`UseMock`, `DontSave`, `ViewedUserId`, `OverriddenUserId`) map onto `Mode` and lose to it when both are set.

**Play-testing Emberfall safely**

```luau
-- A play-test that can never touch a store:
local Data = Scribe({
	Template = template,
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
	Mode = "Mock",
}).Server

-- Inspect Ava's real profile, read-only, as the joining player:
local Data = Scribe({
	Template = template,
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
	Mode = "NoSave",
	TargetUserId = 101,
}).Server
```

| Mode | Reads | Session lock | Saves |
| --- | --- | --- | --- |
| `"Live"` | real profiles and boards | real | yes |
| `"Mock"` | template defaults, in-memory boards | in memory | in memory |
| `"NoSave"` | the real profile as a snapshot | none | never |
| this course | Mock | in memory | in memory, per run |

## Edit mode and storybooks

When `RunService:IsRunning()` is false, in a UI Labs or Hoarcekat storybook, the client half skips the transport and initialises instantly to template defaults; `Observe` and `Changed` work, so a component renders with no server. `Data.Mock(fields, extras)` seeds it: template fields first (derived fields follow from their inputs, so seed `Xp` and read `Level`), then `Perks`, `GiftCredits`, `Leaderboards`, `PurchaseLogs`. `Data.MockCommand(name, fn)` answers `Request` there, with no player argument. Both error outside edit mode.

**Seeding a story**

```luau
-- IsRunning = false is the documented test seam for the engine's edit mode,
-- and what the sandbox uses, since it cannot stop running.
local Data = Scribe({
	Template = template,
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
	IsRunning = false,
}).Client

Data.Mock({ Coins = 1250, Xp = 4200, Settings = { "Music", "Sfx" } }, { Perks = { "VIP" } })
print(Data.Coins.Get(), Data.Level.Get(), Data.Owns("VIP")) --> 1250  5  true

Data.MockCommand("BuyPotion", function(count)
	return true
end)
print(Data.Request("BuyPotion", 3)) --> true
```

## Headless

Scribe's own suite runs headless under Lune against a fake ProfileStore and a stub transport, and the same entry point lets you drive the lifecycle yourself: `Server.build(options, compiled)` returns `(Data, ctx)`, wires nothing to the engine, and `ctx.Persistence.OnPlayerAdded(player)` and `OnPlayerRemoving` drive a session by hand. It is internal, reached through Wally's `_Index`, and carries no compatibility promise.

**A headless session, as the docs show it**

```luau
-- Not runnable in the sandbox, which has no _Index; here Mode = "Mock" and join are the equivalent.
local scribe = ReplicatedStorage.Packages._Index["ericplane_scribe@2.3.0"].scribe
local Template = require(scribe.Template)
local Server = require(scribe.Server)

local compiled = Template.Compile(options.Template)
local Data, ctx = Server.build(options, compiled)

ctx.Persistence.OnPlayerAdded(fakePlayer)
local data = Data.WaitForData(fakePlayer)
data.Coins.Increment(10)
ctx.Persistence.OnPlayerRemoving(fakePlayer)
Data.Stop()
```

**Key points**

- `Mode` is the switch: Live saves, Mock keeps everything in memory, NoSave reads a real profile and never writes.
- Edit mode is the client with no server: instant defaults, `Mock` to seed, `MockCommand` to answer.
- Derived fields follow their seeded inputs, so seed `Xp` and read `Level`.
- `Server.build` drives a headless session for a test runner; it is internal and unversioned.
