# Keeping a game up

*Move players to a new version with Restart Servers, change values without one, and hear about errors on live servers.*

Roblox Advanced · Running a live game · 12 min

---

You fix a bug that duplicates items and publish the fix. Most players still have the bug: a new version reaches only new servers, and every old server runs the old code until it empties.

That is fine for a new hat. A dupe fix cannot wait, and the update guide says the same of any change to your player data's format: restart servers.

## Restarting servers

**Restart Servers**, on the experience's Server Management page in the Creator Dashboard, moves everyone to the new version. The guide recommends ticking *Restart only servers with outdated versions*, so servers already on the new version are left alone.

| Restart Servers | On each old server |
| --- | --- |
| Stops matching players to the old servers | `ServerRestartScheduled` fires, if you chose a delay |
| Waits the delay you chose, up to 60 minutes, or none | Players can leave on their own; save early |
| Teleports each old server's players together to one server on the new version | `BindToClose` runs as the server closes, with its `Enum.CloseReason`, such as `DeveloperUpdate` |

```luau
game.ServerRestartScheduled:Connect(function(restartTime: DateTime, source: Enum.CloseReason, attributes)
	-- one attribute: every client counts down to it, late joiners included
	workspace:SetAttribute("RestartAt", restartTime.UnixTimestamp)
end)

-- On a client
local secondsLeft = workspace:GetAttribute("RestartAt") - workspace:GetServerTimeNow()
```

`restartTime` is the earliest the server will close; it may be a little later, never earlier. `source` is `DeveloperUpdate` for your updates and `RobloxMaintenance` when Roblox restarts servers for maintenance.

> **Recognising older code**
>
> Older games carry a "soft shutdown" script: in `BindToClose` it reserves a server with `TeleportService`, sends everyone there, then back to a fresh public server. Restart Servers now keeps each server's players together and moves them to the new version itself, so new games need none of it.

## Live values and error reports

Not every change needs a new version. **Experience configs** are values you edit on the Creator Hub and publish without one: a drop rate, a switch that turns an event on. The update guide suggests shipping new content early behind a config, then switching it on when the time comes.

```luau
local ConfigService = game:GetService("ConfigService")

local DEFAULT_DROP_RATE = 0.05 -- used if configs never load
local ok, config = pcall(ConfigService.GetConfigAsync, ConfigService)

local function dropRate(): number
	return if ok then config:GetValue("DropRate") or DEFAULT_DROP_RATE else DEFAULT_DROP_RATE
end

if ok then
	config.UpdateAvailable:Connect(function()
		config:Refresh() -- take the new values; a round-based game could wait for the round to end
	end)
end
```

A snapshot holds the values at the moment it was taken. `UpdateAvailable` says newer ones were published; `Refresh` takes them. No restart, no new version.

Errors on live servers are silent unless you collect them. `ScriptContext.Error` fires for every unhandled error, with the message, a stack trace and the script, and `HttpService:RequestAsync` can send it to your own service.

```luau
local HttpService = game:GetService("HttpService")
local ScriptContext = game:GetService("ScriptContext")

local apiKey = HttpService:GetSecret("ErrorsKey") -- from the secrets store, never in code

ScriptContext.Error:Connect(function(message, stackTrace, script)
	pcall(HttpService.RequestAsync, HttpService, {
		Url = "https://errors.example.com/report",
		Method = "POST",
		Headers = { Authorization = apiKey:AddPrefix("Bearer "), ["Content-Type"] = "application/json" },
		Body = HttpService:JSONEncode({ message = message, stack = stackTrace, server = game.JobId }),
	})
end)
```

Secrets are created on the Creator Dashboard and read on live servers; in Studio, add them under Local Secrets. A `Secret` cannot be printed, and goes in the URL or headers, never the body. Turn on **Allow HTTP Requests** first. An error inside a loop fires every frame, so a real reporter batches repeats.

> **Habits for incidents**
>
> - Watch your error reports and the Developer Console after every release.
> - Put new features behind a config switch, so turning one off takes seconds, not a release.
> - Restart with a delay, warn players, and save early.
> - Note what each release changed, so you know what to roll back.
> - After an incident, write down what happened and add the check that would have caught it.

**Key points**

- A published update reaches only new servers. Restart Servers moves everyone, after an optional delay of up to 60 minutes.
- `ServerRestartScheduled` warns each server: tell players when, and save early.
- Experience configs change values without a new version; a snapshot changes only when you `Refresh` it.
- Report errors from `ScriptContext.Error` with `RequestAsync`, and keep the service's key in the secrets store.
