# Organising a place

*One entry Script, ModuleScripts for everything else, each container used for its job, and requires that never go in a circle.*

Roblox Fundamentals · Shipping a place · 11 min

---

By the end of a project, `Main` is 600 lines: coins, lava, checkpoints and the leaderboard, sharing a dozen top-level locals. Changing a coin's value means scrolling past the lava, and one error near the top stops everything below it from connecting.

The docs' recommended layout splits it: **one** `Script` with `RunContext` set to `Server` in ServerScriptService, and a ModuleScript for each job. The Script requires the modules and starts them.

## One entry point, a module per job

| Container | What goes there | Why |
| --- | --- | --- |
| ServerScriptService | The entry `Script` and the server's ModuleScripts | Server only, never sent to clients |
| ReplicatedStorage | ModuleScripts both sides require: config, shared helpers | Replicated to every client, so nothing secret |
| ServerStorage | Templates to `Clone`, maps loaded later | Server only; scripts do not run there |
| Workspace | The world: parts and models, with tags and attributes | Code lives in modules, not inside parts |

**ServerScriptService.Main**

```luau
-- ServerScriptService
--   Main           Script, RunContext = Server
--   Coins          ModuleScript
--   Lava           ModuleScript
-- ReplicatedStorage
--   Config         ModuleScript both sides read
-- ServerStorage
--   CoinTemplate   Part

local Coins = require(script.Parent.Coins)
local Lava = require(script.Parent.Lava)

Coins.start()
Lava.start()
```

`script.Parent` is ServerScriptService, so `require(script.Parent.Coins)` finds the module beside the Script. Every module loads first, then each `start()` connects its events: the shape the docs' "Script locations" guide recommends.

A module runs once per side. The first `require` runs it, and every later `require` on the server gets the same table back. So keep a module's top level to definitions, and put connections and loops in `start()`: requiring is then harmless, and the Script alone decides what starts.

Small modules of plain functions are also easy to test: the Lest course shows how.

## Requires that go in a circle

`Coins` requires `Shop` to refund a purchase, and `Shop` requires `Coins` to charge for one. Neither can finish loading before the other. The docs disagree on the symptom: the ModuleScript reference says the thread hangs with no error, and the modules guide says it errors with `Requested module was required recursively`. The fix is the same: make every dependency point one way.

**ServerScriptService.Shop**

```luau
-- A cycle, which never finishes loading:
--   Coins:  local Shop = require(script.Parent.Shop)
--   Shop:   local Coins = require(script.Parent.Coins)

-- One way: Shop requires Coins, and Coins requires nothing
local Coins = require(script.Parent.Coins)

local Shop = {}

function Shop.buy(player, price)
	if Coins.get(player) >= price then
		Coins.add(player, -price)
		return true
	end
	return false
end

return Shop
```

When two modules need each other, move the shared piece down into a third module they both require, or have the entry Script pass one module to the other's `start()`.

**Key points**

- One `Script` with `RunContext = Server` in ServerScriptService is the entry point; the rest is ModuleScripts.
- ServerScriptService and ServerStorage stay on the server; ReplicatedStorage is for modules both sides require.
- A module runs once per side: keep its top level to definitions and put connections in `start()`.
- `require(script.Parent.X)` finds a module beside the Script.
- Two modules that require each other never finish loading. Make every dependency point one way.
