# Cleaning up

*What leaks on a server that runs for days, and the disconnect-and-clear habits that hold under every setting.*

Roblox Fundamentals · Shipping a place · 11 min

---

A server has been up for three days. Its memory climbs, and a heal that should run once runs 40 times for a player who has spawned 40 times. Nothing ever errored.

That is a leak: work set up for something that is gone, still connected, still in a table, still running.

## What leaks

**A new connection on every spawn**

```luau
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		local humanoid = character:WaitForChild("Humanoid")
		-- Workspace outlives every character, so this connection does too
		workspace:GetAttributeChangedSignal("Speed"):Connect(function()
			humanoid.WalkSpeed = workspace:GetAttribute("Speed")
		end)
	end)
end)
```

Destroying an instance disconnects the connections on *its* signals, so a coin's `Touched` handler goes with the coin. What survives is anything tied to something longer-lived: a connection on Workspace, a service or RunService made per player or character, a `playerData[player]` entry never cleared, a `while true` loop nothing stops.

The habit: keep every connection you make, and disconnect it when what it served is gone.

**Keep it, replace it, clear it**

```luau
local speedConnections = {} -- [Player] = the connection for their current character

local function stopFollowing(player)
	local connection = speedConnections[player]
	if connection then
		connection:Disconnect()
		speedConnections[player] = nil
	end
end

Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		stopFollowing(player) -- the previous character's connection
		local humanoid = character:WaitForChild("Humanoid")
		speedConnections[player] = workspace:GetAttributeChangedSignal("Speed"):Connect(function()
			humanoid.WalkSpeed = workspace:GetAttribute("Speed")
		end)
	end)
end)

Players.PlayerRemoving:Connect(stopFollowing)
```

At most one connection per player, and none once they leave. The `CharacterAdded` connection on the Player is the next paragraph's problem.

`Workspace.PlayerCharacterDestroyBehavior`, set in Studio and not by scripts, decides whether the engine cleans up too. `Enabled` destroys a replaced character and a leaving Player, which disconnects everything on their signals. `Disabled` does not. For `Default`, the docs say only "the engine-default behavior".

So write cleanup that holds under all three: on `PlayerRemoving`, also disconnect what you connected on the Player itself, like its `CharacterAdded`.

## Things that should disappear later

```luau
local Debris = game:GetService("Debris")

-- Destroyed after 3 s, even if this script is disabled or destroyed first
Debris:AddItem(brick, 3)

-- Destroyed after 3 s only if this script is still running then
task.delay(3, brick.Destroy, brick)
```

Neither yields. Debris runs outside your script, which is its guarantee. It holds at most 1,000 items, and past that it destroys the oldest at once.

**Key points**

- A connection lasts as long as the object it is on. Per-character connections on Workspace, services or RunService leak.
- Keep each connection you make, and disconnect the old one before making its replacement.
- On `PlayerRemoving`, disconnect that player's connections, including those on the Player, and clear their table entries.
- `Destroy` ends an instance's own connections, but whether the engine destroys characters and Players depends on a setting. Do not rely on it.
- `Debris:AddItem` destroys later even if your script stops; a `task.delay` callback does not run then.
