# Moderation

*Keep a rule-breaker out with the ban API, not just this server, and log every action so appeals can be answered.*

Roblox Advanced · Running a live game · 11 min

---

A player has flown around your map all afternoon, and a moderator has seen enough. `player:Kick(message)` disconnects them from this server, and that is all: they can rejoin at once, here or on another server.

To keep someone out, ban them with `Players:BanAsync`. Roblox evicts them from every server and refuses their joins until the ban ends.

## Banning

```luau
local Players = game:GetService("Players")

local ok, err = pcall(Players.BanAsync, Players, {
	UserIds = { player.User.Id },
	Duration = 3 * 24 * 60 * 60, -- seconds; -1 bans for good
	DisplayReason = "Exploiting: flying. See the game page to appeal.",
	PrivateReason = "Seen by moderator 1234 flying over the vault, server " .. game.JobId,
})
if not ok then
	warn("Ban failed:", err)
end
```

`DisplayReason` is filtered and shown to the banned player with the time left. `PrivateReason` is never sent to a client. The call makes a web request for each user, so it goes in `pcall`.

| Field | What it does | Default |
| --- | --- | --- |
| `UserIds` | Who, up to 50 at once | Required |
| `Duration` | Seconds; `-1` is permanent | Required |
| `DisplayReason` | Shown to them; up to 400 characters, filtered | Required |
| `PrivateReason` | Your notes; up to 1,000 characters, never shown | Required |
| `ApplyToUniverse` | Every place in the experience, or only this one when `false` | `true` |
| `ExcludeAltAccounts` | `true` spares the user's suspected alternate accounts | `false`: alts are banned too |
| `ApplyDeviceBlock` | Also blocks their device from rejoining for 24 hours (desktop only) | `false` |

Three rules come with it. The ban methods work only while `Players.BanningEnabled` is on, which you switch in the Players service's Properties in Studio. They run only on the server. And bans made in Studio never reach the live game.

`UnbanAsync` lifts a ban, and its device block with it. `GetBanHistoryAsync` pages through a user's past bans, so a repeat offender can get a longer one. It works only on live servers.

> **Tip**
>
> `Kick` still has its uses: disconnecting a player for a reason about this server alone, such as data that failed to load. Rule-breaking gets a ban.

## Keeping a log

The ban guidelines ask you to post your rules, apply them fairly and let players appeal to you directly; Roblox does not mediate. Answering an appeal needs a record of who did what, when and why. Keep one in a DataStore, keyed by the user's ID so a deletion template can match it (module 2), and append with `UpdateAsync` so no entry is lost.

```luau
local log = DataStoreService:GetDataStore("ModerationLog")

local function record(userId: number, entry: { [string]: any }): boolean
	return (pcall(log.UpdateAsync, log, "Player_" .. userId, function(entries, keyInfo)
		entries = entries or {}
		table.insert(entries, entry)
		return entries, { userId }, if keyInfo then keyInfo:GetMetadata() else nil
	end))
end
```

The transform tags the key with the user's ID and keeps its metadata. If another server logs at the same moment, `UpdateAsync` runs the transform again on the newer list, so both entries land.

**Key points**

- `Kick` only disconnects; `BanAsync` keeps a user out of every server until the ban ends.
- `Duration` is in seconds (`-1` is permanent). `DisplayReason` is filtered and shown; `PrivateReason` never leaves your servers.
- By default a ban covers every place in the experience and the user's suspected alternate accounts.
- Switch on `BanningEnabled` in Studio, and call the ban API from the server in `pcall`.
- Log every action, keyed by the user's ID and appended with `UpdateAsync`, so appeals can be answered.
