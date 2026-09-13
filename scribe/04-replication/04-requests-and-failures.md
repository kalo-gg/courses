# Requests and their failures

*Read the third return value, retry with the same key, and know the order a request is checked in.*

Scribe · Replication & Commands · 10 min

---

Every framework failure resolves `Request` as three values: `false`, one sentinel string, and the marker table `Scribe.RequestFailed`. A handler's own `false, "not owned"` looks like the first two values of a refusal, so read the third: it is the marker only when Scribe refused, and nil when the values are the handler's, because a table crossing the wire arrives as a copy. The sentinels are named on `Scribe.RequestReason`: `Timeout`, `BadArgs`, `UnknownCommand` and eight more.

**Telling a refusal from an answer**

```luau
local ok, reason, failed = Data.Request("BuyPotion", "Health", 3)
if failed == Scribe.RequestFailed then
	warn("BuyPotion never ran: " .. reason) -- one of the sentinels
elseif not ok then
	showToast(reason) -- your handler's reason, safe to show a player
end
```

| Sentinel | Raised by | Means |
| --- | --- | --- |
| `rate-limited` | server | over `CommandRateLimit`; the handler never ran |
| `unknown-command` | server | nothing is registered under that name |
| `not-ready` | server | the caller's profile is not Ready |
| `bad-args` | server | the call did not match the Args spec |
| `bad-idempotency-key` | server | the key and the command disagree, or the key is malformed |
| `error` | server | the handler threw; the traceback stays in the server log |
| `reply-encode-failed` | server | the handler returned something the wire cannot carry |
| `timeout` | client | no reply within `RequestTimeout`; the handler may still be running |
| `send-failed` | client | the transport raised while sending |
| `edit-mode`, `mock-error` | client | no server, and no mock or a mock that threw |

## At most once

A retry after a slow network should hand out one daily reward, not two. Mark the command `Idempotent = true` and call it with `Data.RequestOnce(name, key, …)`: the server runs the handler at most once per key and answers every repeat with the original reply. Generate the key once per intent, not per attempt. Keys are at most 64 bytes, live for the session, and are scoped to the command name. A key on a command that is not idempotent, or a plain `Request` on one that is, is refused with `bad-idempotency-key`.

**ClaimDaily, and the retry that cannot double-grant**

```luau
-- server
Data.Command("ClaimDaily", { Idempotent = true }, function(player)
	local data = Data[player]
	if data.LastDaily.Active() then
		return false, "already claimed today"
	end
	data.LastDaily.SetTimed(true, 86400) -- clears itself after a day
	data.Gems.Increment(100)
	return true
end)

-- client
local claimId = HttpService:GenerateGUID(false)
local function claim()
	return Data.RequestOnce("ClaimDaily", claimId)
end

local ok, reason, failed = claim()
if failed == Scribe.RequestFailed and reason == Scribe.RequestReason.Timeout then
	ok, reason = claim() -- same key, so the server replays its first answer
end
```

## Rate limits

Each player gets a token bucket sized and refilled at `CommandRateLimit` per second, 20 by default: a burst up to the full allowance passes, then it drains to the sustained rate. Over the limit the reply is `false, "rate-limited"`. After 256 rate-limited frames with no ten-second gap, even the rejection reply stops. `not-ready` is uniform across every name, so nobody can enumerate your commands by diffing replies.

**Two refusals and one replay, seen from the client**

```luau
local ok, reason, failed = Client.Request("Nope")
print(ok, reason, failed == Scribe.RequestFailed) --> false  unknown-command  true

print(Client.RequestOnce("ClaimDaily", "intent-1")) --> true
print(Client.RequestOnce("ClaimDaily", "intent-1")) --> true, replayed; gems still 100
print(Scribe.GetMetrics().CommandsDeduped) --> 1
```

**Key points**

- A framework refusal is `false, sentinel, Scribe.RequestFailed`; a handler's values never carry the marker.
- `Idempotent = true` plus `RequestOnce(name, key)` runs a handler at most once per key; keys are per intent.
- Requests pass a fixed gauntlet: rate limit, ready gate, registry, arity, types, handler, reply.
- The rate limit is a token bucket of `CommandRateLimit` per second, 20 by default.
