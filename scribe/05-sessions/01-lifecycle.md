# The session lifecycle

*WaitForData and its seven reasons, the hook that runs before Ready, the hook that runs before the last save, and the signals.*

Scribe · Sessions & Saving · 10 min

---

A profile does not exist the instant a player joins: Scribe fetches it, reconciles it against the template and takes the session lock. `WaitForData(player, timeout?)` yields up to 60 seconds and returns `(accessor?, reason?)`. `reason` is one of seven strings, named on `Scribe.Reason`; only `StillLoading` is worth a retry: the wait elapsed while the load was in flight. `Data.GetState(player)` answers `Loading`, `Ready` or `SessionEnded` without yielding.

**The handler, with the one retry that helps**

```luau
Players.PlayerAdded:Connect(function(player)
	local data, reason = Data.WaitForData(player)
	if not data and reason == Scribe.Reason.StillLoading then
		data, reason = Data.WaitForData(player, 60) -- still coming: give it the rest of the load window
	end
	if not data then
		if reason ~= Scribe.Reason.PlayerLeft then
			warn("Emberfall: no data for " .. player.Name .. ", " .. reason)
		end
		return
	end

	local level = Instance.new("IntValue")
	level.Name = "Level"
	data.Level.Observe(function(value)
		level.Value = value
	end)
end)
```

| Reason | `Scribe.Reason` | Meaning | Retry? |
| --- | --- | --- | --- |
| `player-left` | `PlayerLeft` | The player left; by far the most common, and not an error | no |
| `still-loading` | `StillLoading` | The wait elapsed while the load was still in flight | yes |
| `timeout` | `Timeout` | The wait elapsed and Scribe has no session for this player | no |
| `load-failed` | `LoadFailed` | The profile could not be loaded | no |
| `migration-failed` | `MigrationFailed` | A migration step threw; the profile was released unmigrated | no |
| `session-ended` | `SessionEnded` | The session ended while the player was still in game | no |
| `shutdown` | `Shutdown` | The server is closing | no |

## Hooks at both ends

`OnPlayerInit(player, rawData, isNewProfile)` runs once per player after the profile loads and before it is Ready, against the **raw** table: writes bypass validation, derived fields are not in it, a Flags field is its list of enabled names. `isNewProfile` is true for a new profile, a reset, and a first-session crash recovery.

`OnPlayerLeaving(player, data, reason)` is the counterpart: the typed accessor, before the final save, so what it writes persists. It must not yield, and it is skipped when the profile never reached Ready or another server stole the session: best-effort bookkeeping.

**A starter purse and a playtime counter**

```luau
local joinedAt = {}

Scribe({
	Template = template,
	ProfileStoreIndex = "EmberfallPlayerData",
	ProfileKeyPrefix = "PLAYER_",

	OnPlayerInit = function(player, rawData, isNewProfile)
		joinedAt[player.UserId] = os.time()
		if isNewProfile then
			rawData.Coins = 500 -- Emberfall's starter purse
			rawData.Settings = { "Music", "Sfx" } -- a Flags field is its enabled names
		end
	end,

	OnPlayerLeaving = function(player, data, reason)
		data.Stats.Playtime.Increment(os.time() - joinedAt[player.UserId])
	end,
})
```

## Signals

`OnSave` and `SessionEnded` are signals, like every `On…` member on `Data`, `Client` and `Scribe`: `:Connect(fn)` returns a connection with `:Disconnect()`, `:Once` runs once, `:Wait` yields for the next fire. Every handler runs on its own thread, and a handler that errors is reported without stopping the signal. With `KickOnSessionEnd = true`, the default, a stolen session also kicks the player so their client cannot act on stale data.

**Connect, Once, Wait**

```luau
Data.SessionEnded:Connect(function(player, reason)
	if reason == Scribe.Reason.SessionEnded then
		analytics:Log("session_stolen", player.UserId)
	end
end)

Data.OnSave:Once(function(info)
	print("first save took", info.Duration)
end)

local info = Data.OnSave:Wait() -- yields for the next one
```

**Key points**

- `WaitForData` answers `(accessor?, reason?)`; only `Scribe.Reason.StillLoading` is worth a retry.
- `OnPlayerInit` sees the raw table before Ready; `isNewProfile` is your starter-kit switch.
- `OnPlayerLeaving` runs before the final save with the accessor and must not yield.
- `OnSave` and `SessionEnded` are signals: Connect, Once, Wait, one thread per handler.
- A stolen session kicks by default, so no client acts on stale data.
