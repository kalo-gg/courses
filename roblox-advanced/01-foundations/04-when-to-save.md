# When to save

*Save when a player leaves, on a timer, and when the server closes, and wait for every save before it does.*

Roblox Advanced · DataStores · 11 min

---

Your game saves on `PlayerRemoving`, and every test passes. Then you ship an update, Roblox closes the old servers, and players on them can lose their last session. A closing server does not wait for those saves: it gives your code one hook, `BindToClose`, and 30 seconds.

A game saves at three moments. This lesson covers each one and the failure it protects against.

## Leaving, and a timer

```luau
local AUTOSAVE_INTERVAL = 60
local sessions: { [Player]: { Coins: number } } = {} -- players whose load succeeded

Players.PlayerRemoving:Connect(function(player)
	local data = sessions[player]
	sessions[player] = nil
	if data then
		save(player, data)
	end
end)

task.spawn(function()
	while true do
		task.wait(AUTOSAVE_INTERVAL)
		for player, data in sessions do
			task.spawn(save, player, data)
		end
	end
end)
```

Only players whose load succeeded are in `sessions`, so a failed load is never saved over. The autosave spawns each save, so one slow request does not hold up the rest.

How often to autosave is a trade. Every save spends from the write budget of lesson 3, and the interval is the most a crash can cost. The docs' player data guide saves every few minutes, and starts each player's first autosave at a random point in the interval, so servers that started together do not all write at once.

## BindToClose

`game:BindToClose(fn)` runs `fn` before the server shuts down and passes it an `Enum.CloseReason`, such as `DeveloperUpdate` or `ServerEmpty`. Every bound function runs at the same time, and the server waits for them for at most 30 seconds, then shuts down whatever is still running.

So start every save at once, and return only when they have all finished, including saves `PlayerRemoving` started a moment ago.

```luau
local pending = 0 -- saves in flight, wherever they started

local function save(player: Player, data)
	pending += 1
	-- the UpdateAsync write from lesson 2, retried as in lesson 3
	pending -= 1
end

game:BindToClose(function(reason: Enum.CloseReason)
	for player, data in sessions do
		task.spawn(save, player, data)
	end
	while pending > 0 do
		task.wait()
	end
end)
```

`task.spawn` runs each save until its first yield, so `pending` has already counted them all when the wait loop starts.

| Moment | What it protects | What it cannot do |
| --- | --- | --- |
| `PlayerRemoving` | A player who leaves a server that keeps running | Finish if the server is closing |
| Autosave loop | Players on a server that crashes | Save anything after its last run |
| `BindToClose` | Players still in the server when it closes | Run past 30 seconds, or run at all after a crash |

> **Recognising older code**
>
> `game.OnClose = function() … end` is the deprecated form of `BindToClose`, and an autosave written `spawn(function() while wait(60) do … end end)` uses two deprecated globals. Write `game:BindToClose`, and `task.spawn` with `task.wait`. When a Studio test stops, it calls your `BindToClose` functions too; the docs suggest checking `RunService:IsStudio()` in them so an offline test does not run the whole shutdown save.

**Key points**

- Save on `PlayerRemoving`, on an autosave loop, and in `BindToClose`: each covers a failure the others miss.
- Keep only players whose load succeeded in `sessions`, so a failed load is never saved over.
- Every `BindToClose` function runs at once, and the server waits for them for at most 30 seconds.
- In `BindToClose`, start every save at once and return only when every save in flight has finished.
