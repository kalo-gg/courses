# Timers and cooldowns

*A Timed field is a value that expires; a cooldown is a gate with no value. Picking the wrong one is the common mistake.*

Scribe · Sessions & Saving · 11 min

---

Two questions come up constantly: "is this buff still running?" and "can this happen again yet?". `Scribe.Timed(default)` answers the first: `SetTimed(value, seconds)` sets the value and arms a timer, and when it lapses a once-per-second sweep writes the declared default back and fires `Changed`; `Active()` returns `(active, remaining)`. Durations floor to one second and cap around 126 years, so `SetTimed(v, math.huge)` means permanent. A plain `Set` does not cancel a running timer.

**The daily reward**

```luau
local function claimDaily(player)
	local data = Data[player]
	if data.LastDaily.Active() then
		return false -- already claimed today
	end
	data.Coins.Increment(250, {
		TransactionType = Enum.AnalyticsEconomyTransactionType.TimedReward,
		ItemSku = "DailyReward",
	})
	data.LastDaily.SetTimed(true, 86400) -- clears itself back to false after a day
	return true
end

-- client: the countdown comes for free
Data.LastDaily.Observe(function(spent)
	claimButton.Visible = not spent
end)
```

## Extending

Buying a second boost while the first runs should add time: `ExtendTimed(seconds)` adds to what is left. Check `Active()` first, because `ExtendTimed` with no timer running arms a fresh one from now, so a field that was never `SetTimed` then counts down and reverts to its default. It never writes the value, so it has nothing to replicate. A timed field cannot feed a derived field and cannot live inside an element shape.

**XpBoost: extend or arm**

```luau
if data.XpBoost.Active() then
	data.XpBoost.ExtendTimed(1800) -- add 30 minutes to what is left
else
	data.XpBoost.SetTimed(true, 1800) -- arm a fresh 30 minutes
end

local active, remaining = data.XpBoost.Active() --> true, 1800
```

|  | `Scribe.Timed` field | Cooldown |
| --- | --- | --- |
| Holds a value | yes | no |
| Declared in the template | yes | no, keyed by a string |
| Replicates to the owner | yes, like any field | never |
| Expiry notification | `Changed` fires with the default | `OnCooldownEnded` |
| Read without side effects | `Get()`, `Active()` | `PeekCooldown` |
| Good for | boosters, buffs, a claimed flag | ability recharges, rate limits |

## Cooldowns

A cooldown has no declarator and no field: a server-side timer keyed by any string. `Data.OnCooldown(player, key, seconds)` **checks and arms in one call**, answering true while one is running, so call it only at the moment of use. `PeekCooldown` reads `(onCooldown, remaining)` without arming, for UI. `ClearCooldown` resets. `Data.OnCooldownEnded` fires `(player, key)` within about a second of expiry, and not for cooldowns that lapsed offline. They live in the profile, survive rejoins, and never replicate. Keys starting with `@` are Scribe's.

**Dash**

```luau
-- OnCooldown CHECKS and ARMS in one call, so call it only at the moment of use.
if not Data.OnCooldown(player, "Dash", 12) then
	performDash(player) -- it was off; a fresh 12 s cooldown is now running
end

local onCooldown, remaining = Data.PeekCooldown(player, "Dash") -- read-only, safe for UI
Data.ClearCooldown(player, "Dash") -- support and testing reset

Data.OnCooldownEnded:Connect(function(player, key)
	if key == "Dash" then
		tellClientDashIsReady(player)
	end
end)
```

**Key points**

- `Scribe.Timed` holds a value that expires; `SetTimed` arms it, `Active()` reads it, the lapse writes the default and fires `Changed`.
- A plain `Set` never cancels a timer; `ExtendTimed` adds time, and arms a fresh timer if none is running.
- A cooldown is a keyed server-side gate: `OnCooldown` checks and arms, `PeekCooldown` only reads, `ClearCooldown` resets.
- `OnCooldownEnded` is the only expiry notification a cooldown gets, within about a second.
- Timed for state a UI shows; a cooldown for a gate nothing reads.
