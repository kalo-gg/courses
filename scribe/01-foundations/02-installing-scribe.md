# Installing Scribe 2.3

*One Wally line, one shared module both realms require, and the Studio setting that decides whether anything saves.*

Scribe · Scribe Foundations · 9 min

---

With Wally, Scribe is one line: `Scribe = "ericplane/scribe@2.3.0"` in the shared realm. ProfileStore is vendored, so nothing appears in `ServerPackages`. Two other routes exist: the model on the Creator Store, and Scribe Studio, a plugin that renders the library's diagnostics live. 2.3.0 was released on 4 September 2026.

**wally.toml**

```toml
[dependencies]
Scribe = "ericplane/scribe@2.3.0"
```

## One shared module, two halves

`Scribe(options)` returns a **bundle**: `{ Server, Client }`. A server Script uses `.Server`; a LocalScript uses `.Client`. `RunService:IsServer()` decides which half is real; touching the other errors with a message saying so.

Requiring on the client is what starts replication. Leave it out and the server warns `CLIENT_HANDSHAKE_TIMEOUT` after ten seconds, and every client read stays at its template default.

**The shared module, and the two requires**

```luau
-- ReplicatedStorage/Shared/EmberfallData.luau
local Scribe = require(game:GetService("ReplicatedStorage").Packages.Scribe)

return Scribe({
	Template = { Coins = Scribe.Int(0, { Min = 0 }) },
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
})

-- ServerScriptService/Emberfall.server.luau
local Data = require(ReplicatedStorage.Shared.EmberfallData).Server

-- StarterPlayerScripts/Hud.client.luau
local Data = require(ReplicatedStorage.Shared.EmberfallData).Client
```

## Studio, DataStores and Mock mode

Studio cannot reach DataStores until the place is published and **Enable Studio Access to API Services** is on. With it off, ProfileStore quietly uses an in-memory store: a play-test looks fine and nothing survives a restart.

When you *want* that isolation, ask for it: `Mode = "Mock"`. This whole course runs in Mock mode, so nothing here can touch a real store. `Banner = false` silences the startup line; `Scribe.Version` is `"2.3.0"`.

> **The tell is one line in the Output**
>
> `[ProfileStore]: Roblox API services unavailable - data will not be saved`. If a play-test that should be saving prints it, check the setting before you debug anything else.

**Check what you are running**

```luau
local Scribe = require(game:GetService("ReplicatedStorage").Packages.Scribe)

print(Scribe.Version) --> 2.3.0

local bundle = Scribe({
	Template = { Coins = 0 },
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",
})
--> Running Scribe v2.3.0 by ericplane. Server is now initialized
```

The banner is Scribe's own line, printed once per bundle. The sandbox prints it too.

**Key points**

- Install with Wally (`ericplane/scribe@2.3.0`); ProfileStore comes inside.
- One shared ModuleScript returns the bundle; the server uses `.Server`, the client `.Client`.
- The client's require is what starts replication; without it the server warns CLIENT_HANDSHAKE_TIMEOUT.
- `Mode = "Mock"` is how you play-test without a real store; the sandbox always runs that way.
