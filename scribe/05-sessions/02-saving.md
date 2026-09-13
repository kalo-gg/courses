# Saving

*The autosave, the save on leave, Flush and what its false really means, and the two calls that watch save state.*

Scribe · Sessions & Saving · 10 min

---

Scribe autosaves every `SaveInterval` seconds, 300 by default and floored at 15; it sets ProfileStore's global period, so every bundle shares it, and `Scribe.Configure({ AutoSaveInterval = 60 })` is the process-wide spelling. It also saves when a player leaves and on `BindToClose`. For a grant you cannot lose, `Data.Flush(player, { Force?, Timeout? })` yields until the save is confirmed and returns whether it landed; it costs nothing when the profile is already clean.

**Flush after a grant, and watch the saves**

```luau
Data.Purchase(player, spec)
Data.Flush(player, { Force = true }) -- persist immediately, even past a blocked wipe guard

Data.OnSave:Connect(function(info)
	-- { Player, Ok, Duration, At }
end)

local info = Data.GetSaveInfo(player) -- { LastSaveAt, LastResult, Dirty, Size }
```

## Watching the autosave happen

In the sandbox time moves when you wait, so `SaveInterval = 30` and `task.wait(31)` shows the autosave: `GetSaveInfo(ava).Dirty` goes true after a write and false after the save, and `OnSave` fires with `Ok = true`. This is the first lesson that waits past ten seconds, so the `CLIENT_HANDSHAKE_TIMEOUT` warning appears; it is the sandbox's missing client, not a save problem.

**Dirty, flushed, autosaved**

```luau
data.Coins.Increment(5)
print(Data.GetSaveInfo(ava).Dirty) --> true
print(Data.Flush(ava)) --> true, and OnSave fires with Ok = true
print(Data.GetSaveInfo(ava).Dirty) --> false

data.Coins.Increment(5)
task.wait(31) -- the autosave lands in here
print(Data.GetSaveInfo(ava).Dirty, Data.GetSaveInfo(ava).LastResult) --> false  Ok
```

| Call | When | Returns |
| --- | --- | --- |
| `Flush(player)` | now; yields up to 15 s | true once confirmed |
| `Flush(player, { Force = true })` | now, even past a blocked wipe guard | the same |
| `GetSaveInfo(player)` | any time, no yield | `{ LastSaveAt, LastResult, Dirty, Size }` |
| `OnSave` | after every save | `{ Player, Ok, Duration, At }` |
| the autosave | every `SaveInterval` seconds | nothing |
| the leave save | after `OnPlayerLeaving` | nothing |

> **Never re-grant on a false Flush**
>
> Re-credit the coins, have the save land after all, and the player owns the item and the coins. A false means unresolved: retry the same idempotent operation, or show a pending state and let the next load tell you the truth.

## Stopping a bundle

A bundle normally lives for the whole server. `Data.Stop()` releases everything one holds: the timed sweep, the leaderboard loops, the per-frame flush, the Players and MarketplaceService listeners, and the transport channel, which is what lets a second bundle bind the Default channel in the same process. It is idempotent and it does **not** save or end sessions, so flush first if the data matters, and let players leave before a second bundle loads them, or their profiles are still locked by the first.

**Stop does not save**

```luau
Data.Flush(player, { Force = true })
leave(player) -- in the sandbox; on a real server the session ends with the player
Data.Stop()

local Data2 = Scribe({ … }).Server -- the Default channel is free again
```

**Key points**

- Saves happen every `SaveInterval`, on leave, on shutdown, and when you `Flush`.
- `Flush` yields for confirmation and is free when the profile is clean; a false is unresolved, never a reason to re-grant.
- `OnSave` reports every save; `GetSaveInfo` reads a player's save state without yielding.
- `Data.Stop()` releases a bundle's loops and channel and saves nothing.
