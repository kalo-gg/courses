# Laying out Emberfall

*A shared module, server services, a client HUD, a diagnostics script and an admin path: the plan the final project follows.*

Scribe · Shipping Emberfall · 11 min

---

Everything in the course fits in five places. The shared module owns the template, the options and the two hooks; both realms build their bundle from it and nothing else declares a field. Server services export functions that take `Data` (or a player's accessor) and register commands; they own no state. The client HUD is `Observe` calls and `Request`s. A diagnostics script connects `OnIssue` and `OnStatusChanged`. An admin path is a command with a permission check in front of the offline operations.

**Emberfall's tree**

```text
ReplicatedStorage
├── Packages/Scribe                    (Wally)
└── Shared/EmberfallOptions            the template, the options, the hooks
ServerScriptService
├── Emberfall.server.luau              builds Data, calls each service's register
├── Services/Inventory.luau            functions over a player's accessor
├── Services/Commands.luau             Data.Command registrations
├── Services/Admin.luau                the admin path
└── EmberfallDiagnostics.server.luau   OnIssue, OnStatusChanged, a sink
StarterPlayer/StarterPlayerScripts
└── Hud.client.luau                    Observe and Request
```

## Naming the accessor type

To hold a player's tree in your own table you need its type: `Scribe.PlayerData<T>` where `T` is `typeof(template)`, and `Scribe.ServerData<T>` / `Scribe.ClientData<T>` for the whole `Data` object. On a large template the `Scribe({ … })` call can hit "Code is too complex to typecheck"; the escape hatch is to name the options as `Scribe.ScribeOptions<typeof(template)>`, cast the call, and annotate the half you use. The typed tree needs the new Luau type solver (Studio's beta, or your LSP's setting); Scribe runs the same without it, and `Data.Raw` is the untyped escape hatch.

**Types for a service**

```luau
export type Template = typeof(template)
type EmberfallData = Scribe.PlayerData<Template>

local function newSession(player: Player, data: EmberfallData)
	data.Stats.Sessions.Increment(1)
end

-- When the call itself is too complex to typecheck:
local options: Scribe.ScribeOptions<Template> = { Template = template, … }
local Data = (Scribe :: any)(options).Server :: Scribe.ServerData<Template>
```

## The plan for the final project

The final project is this tree in the sandbox: the options module from two projects, Project 1's inventory service unchanged, Project 2's commands to extend, a diagnostics module, a HUD run through `asClient`, and an admin command that grants offline. New in the template: a `Shared` nameplate, a `ServerOnly` strike counter, and a migration step, because the template changed shape since Project 2 wrote profiles.

**The admin path**

```luau
local ADMINS = { [101] = true }

Data.Command("AdminGrantGems", {
	Args = { "number", Scribe.Int(1, { Min = 1, Max = 1000 }) },
}, function(player, targetUserId, amount)
	if not ADMINS[player.UserId] then
		return false, "not allowed"
	end
	local target = Players:GetPlayerByUserId(targetUserId)
	if target and Data.GetState(target) == "Ready" then
		Data[target].Gems.Increment(amount)
		return true
	end
	return Data.UpdateOffline(targetUserId, function(raw)
		raw.Gems = (raw.Gems or 0) + amount
	end)
end)
-- The user id is an argument here BECAUSE the caller's own identity was checked first.
-- That is the one place it is allowed.
```

> **Where to go next**
>
> The guides this course did not cover: Gifting (escrowed on the buyer's profile), Exchange (two players, one server), Economy Analytics (the tag tables), Custom Transports, the UI adapters (`useScribe` for Fusion, React and Vide in the repository's `adapters/`), and Scribe Studio, the plugin that renders the diagnostics live.

**Key points**

- Declare once, in the shared module; nothing else names a field.
- Wait, then read and write through accessors; services take Data and own no state.
- The client observes, and asks with a command whose Args prove the shape.
- A transaction, then a flush, for anything the player would notice losing.
- Test in Mock, ship with diagnostics on, and put every admin action behind a permission check.
