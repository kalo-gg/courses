# Diagnostics

*Stable log codes, a health status, per-player save state and counters: what to route where, and how to read them.*

Scribe · Shipping Emberfall · 10 min

---

Every failure has a stable code that never changes between versions, so it is safe to match on. `Scribe.OnIssue` fires for every `Error` and `Fatal` entry; `Scribe.OnStatusChanged` fires only when the health verdict moves between `Healthy`, `Degraded` and `Outage`; `Scribe.GetStatus()` reads it. `Degraded` changes nothing on its own; only `Outage` gates anything inside Scribe, and only receipts, gifts, restores and erases.

**ServerScriptService/EmberfallDiagnostics.server.luau**

```luau
local Scribe = require(game:GetService("ReplicatedStorage").Packages.Scribe)

Scribe.OnIssue:Connect(function(entry)
	warn("Scribe issue:", entry.Code, entry.Message)
end)

Scribe.OnStatusChanged:Connect(function(status, previous)
	warn("Scribe status:", previous, "->", status)
	if status == "Outage" then
		-- pause the shop, show a banner; receipts are refused until it clears
	end
end)
```

## The ring and the console

Every entry is `{ At, Level, Category, Code, Message, Context }`. Entries land in a ring of 512 (`LogRingSize`) whatever their level; `LogLevel` decides only what reaches the console (`"Warn"` on a live server, `"Debug"` in Studio; the sandbox is not Studio, so it prints Warn and above). `Scribe.GetRecentLogs({ Code?, Level?, Category?, Limit? })` reads the ring oldest-first; `Scribe.AddLogSink(fn)` forwards entries as they happen and returns a detach function, which you keep when the sink has a lifetime.

**A sink, and a filtered read**

```luau
local seen = {}
local detach = Scribe.AddLogSink(function(entry)
	if entry.Level == "Warn" then
		table.insert(seen, entry.Code)
	end
end)

data.Coins.Set(-10) -- a server-side write the template refuses: clamped, and logged
print(table.concat(seen, ",")) --> ANOMALY

for _, entry in Scribe.GetRecentLogs({ Code = "ANOMALY", Limit = 20 }) do
	print(entry.Context.Path, entry.Context.Reason) --> Coins  OutOfBounds
end
detach()
```

| Metric | Kind | Worth a panel because |
| --- | --- | --- |
| `ActiveSessions` | gauge | the load the rest is measured against |
| `ProfilesLoaded` / `ProfileLoadFailures` | counters | a rising failure rate is the first sign of a store problem |
| `SavesOk` / `SavesFailed` | counters | the durability of everything else |
| `SaveDuration` | distribution | `{ Count, Average, Max }`; the tail is in `GetPercentiles` |
| `ProfileSize` | distribution | `PROFILE_SIZE` warns near the 4 MB ceiling |
| `DataStoreErrors` | counter family | split by class as `DataStoreErrors_<Class>` |
| `WipeGuardTrips` / `Anomalies` | counters | saves that looked like loss, writes the template refused |

## Counters

`Scribe.GetMetrics()` is a flat snapshot: counters and gauges are numbers, timing and size distributions are `{ Count, Average, Max }`; a key exists only once something emitted it, so a missing key is zero. `Scribe.GetPercentiles()` gives `{ P50, P90, P99 }` over the last 256 samples, which is what moves when one key goes slow while the average does not. `DataStoreErrors_<Class>` splits errors into `Throttled`, `Failed`, `Unresolved` and `Rejected`, a reporting dimension: only `Throttled` changes what Scribe does.

**Reading the counters**

```luau
local m = Scribe.GetMetrics()
print(m.ProfilesLoaded, m.SavesOk or 0, m.Anomalies or 0) --> 1  0  1

local p = Scribe.GetPercentiles().SaveDuration
if p then
	print(p.P50, p.P99)
end
```

**Key points**

- Codes are stable; match on them. `OnIssue` is Error and Fatal, `OnStatusChanged` is the health verdict.
- Degraded reports; only Outage refuses, and only receipts, gifts, restores and erases.
- The ring keeps 512 entries of every level; `LogLevel` only filters the console.
- `AddLogSink` forwards live entries and returns a detach; `GetRecentLogs` filters the ring.
- `GetMetrics` is flat and sparse; `GetPercentiles` shows the tail the average hides.
