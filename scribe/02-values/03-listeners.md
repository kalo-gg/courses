# Observe and Changed

*Observe runs now and on every change; Changed only on the transition. Both stop with the function they return.*

Scribe · Fields & Values · 10 min

---

`Observe(fn)` calls you immediately with the current value and again on every change, which removes the question of what to show before the first value arrives. `Changed(fn)` skips that first call and hands you `(new, old)`, so it is the one to reach for when only the transition matters. Both return a function that disconnects the listener.

**UI wants Observe; transitions want Changed**

```luau
-- client
local stop = Data.Coins.Observe(function(coins)
	coinsLabel.Text = tostring(coins)
end)

Data.Level.Changed(function(new, old)
	if new > old then
		playLevelUpEffect(new)
	end
end)
```

## Stopping, and what does not fire

A write that leaves the value unchanged fires nothing and sends nothing, and that includes a clamp that lands on the current value and an `Increment(0)`. The returned function disconnects one listener. A listener made inside a `PlayerAdded` handler needs no cleanup of its own: it belongs to that player's accessor tree and goes with the session.

**Disconnect, and a silent write**

```luau
local stop = data.Coins.Changed(function(new, old)
	print(old, "->", new)
end)

data.Coins.Set(40) --> 0 -> 40
data.Coins.Set(40) -- nothing: the value did not change
data.Coins.Increment(0) -- nothing
stop()
data.Coins.Set(50) -- nothing: disconnected
```

## Containers fire differently

On a record or a container, `Changed` reports state rather than a transition: it takes `(new, old)`, `old` is usually the same table as `new`, and it is **coalesced**: a `Data.Batch` writing four fields under `Stats` fires `Stats.Changed` once. To learn which child moved, the container family (`OnChildChanged`, `OnKeyAdded`, `OnKeyRemoved`) is the last lesson of the Containers module.

Handlers must not yield, on any listener.

**One batch, one fire**

```luau
local fires = 0
data.Stats.Changed(function()
	fires += 1
end)

Data.Batch(ava, function()
	data.Stats.Deaths.Increment(1)
	data.Stats.Playtime.Increment(60)
end)

print(fires) --> 1
```

**Key points**

- `Observe` fires now and on every change; `Changed` fires on the transition with `(new, old)`.
- A write that changes nothing fires nothing: same value, a clamp onto the current value, `Increment(0)`.
- Both return a function that disconnects the listener.
- Container `Changed` is coalesced and reports state; leaf `Changed` is a transition.
