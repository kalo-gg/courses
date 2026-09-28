# Player state and teams

*leaderstats for what players see, attributes for what other scripts read, tables for the rest, and the Teams service.*

Roblox Fundamentals · Players · 11 min

---

A coin script keeps `coins[player] = 0` in a local table. The number is right, but the leaderboard in the corner of the screen never shows it, no other script can read it, and after the player leaves, their entry is still in the table.

Per-player state has three homes, and each suits a different reader.

## leaderstats

```luau
local function onPlayerAdded(player: Player)
	local leaderstats = Instance.new("Folder")
	leaderstats.Name = "leaderstats"

	local coins = Instance.new("IntValue")
	coins.Name = "Coins"
	coins.Value = 0
	coins.Parent = leaderstats

	leaderstats.Parent = player
end

-- later, anywhere on the server
player.leaderstats.Coins.Value += 10
```

The Folder must be named exactly `leaderstats`, in lowercase. Each IntValue, NumberValue or StringValue inside it is a column named after the value.

The other two homes:

- **An attribute on the Player**, such as `player:SetAttribute("Stage", 3)`, for a value other scripts read and can watch with `GetAttributeChangedSignal`.
- **A table in your script, keyed by the Player**, for what only that script needs, including things no attribute can hold, such as a connection. Clear the entry in `PlayerRemoving`, or it outlives the player.

```luau
player:SetAttribute("Stage", 1)
print(player:GetAttribute("Stage")) --> 1

local lastHeal: { [Player]: number } = {}
lastHeal[player] = os.clock()

Players.PlayerRemoving:Connect(function(player)
	lastHeal[player] = nil
end)
```

A rejoin is a new Player object, so a table keyed by the Player lasts one session. What must outlive it is keyed by `player.User.Id` and saved, which Roblox Advanced covers.

## Teams

Teams live in the `Teams` service as `Team` objects, each with its own `TeamColor`. A player's team is `player.Team`, and setting it moves them.

When a player joins, the engine puts them on the `AutoAssignable` team with the fewest players, or leaves them neutral if no team is auto-assignable. It never rebalances when players leave, so a game that wants even teams writes that itself.

```luau
local Teams = game:GetService("Teams")

local red = Instance.new("Team")
red.Name = "Red"
red.TeamColor = BrickColor.new("Bright red")
red.AutoAssignable = false
red.Parent = Teams

player.Team = red
print(#red:GetPlayers()) --> 1
```

Give every team its own colour: spawns and much of the built-in team behaviour go by `TeamColor`.

**Key points**

- A Folder named exactly `leaderstats` under the Player, holding Value objects, is what the leaderboard shows.
- Attributes on the Player are for values other scripts read and watch.
- A table keyed by the Player is private to one script; clear it in `PlayerRemoving`.
- `Team` objects live in `Teams`; `player.Team` moves a player, and `AutoAssignable` evens teams only as players join.
