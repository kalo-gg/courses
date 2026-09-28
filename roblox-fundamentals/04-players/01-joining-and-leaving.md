# Joining and leaving

*PlayerAdded and PlayerRemoving, the players who were there before your script, and which name to trust.*

Roblox Fundamentals · Players · 10 min

---

A welcome script greets everyone who joins. On a live server it works. Start a Test in Studio and your own player often goes ungreeted, with no error.

The script is not wrong, only late. This lesson covers the two events on the `Players` service, the players who arrive before your script does, and which of a player's names to trust.

## PlayerAdded and PlayerRemoving

```luau
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
	print(player.DisplayName .. " joined")
end)

Players.PlayerRemoving:Connect(function(player)
	print(player.DisplayName .. " is leaving")
end)
```

`PlayerRemoving` fires just before the player leaves, so the Player and everything under it are still there to read, save or clear.

Here is why the greeting missed you. In a solo playtest, the docs say, the player is created *before* scripts run, so the `PlayerAdded` that announced you fired while nothing was listening. A script added to a running server has the same problem.

The fix: put the handler in a named function, connect it, then call it for every player already in `Players:GetPlayers()`.

**Handle the players who are already here**

```luau
local Players = game:GetService("Players")

local function onPlayerAdded(player: Player)
	print("Welcome, " .. player.DisplayName .. "!")
end

Players.PlayerAdded:Connect(onPlayerAdded)
for _, player in Players:GetPlayers() do
	task.spawn(onPlayerAdded, player)
end
```

`task.spawn` gives each call its own thread, so a handler that waits for something never holds up the next player. Nothing yields between the connection and the loop, so each player is handled exactly once.

## Which name to trust

A player has three identities. `Name` is the account's username, `DisplayName` is what other players see, and `User` is who they are in your game: a `User` value whose `Id` is a number.

Names can change; `User.Id` does not. So anything that must still match the player after a rename or a rejoin is keyed by `player.User.Id`, never by a name. Show people the `DisplayName`.

| Property | Example | Unique? | Use it for |
| --- | --- | --- | --- |
| `User.Id` | `2918475` | Yes, and it never changes | Keys: records, saved data, ban lists |
| `Name` | `ada_builds` | Yes, but the account can be renamed | Logs, and finding a player by username |
| `DisplayName` | `Ada` | No: two players can share one | Anything players read |

> **Recognising older code**
>
> `game.Players.PlayerAdded:connect(function(plr) … plr.userId … end)` is older code. It uses camelCase names, which the docs say are mostly deprecated (`userId` is). Write `Players.PlayerAdded:Connect(…)`, with `Players` from `GetService`. `player.UserId` still works, but the docs now name `Player.User` the standard identifier in new code.

**Key points**

- `PlayerAdded` fires as a player enters; `PlayerRemoving` fires just before they leave, while their objects can still be read.
- Players who joined before your script connected never fire `PlayerAdded` for it: connect, then loop over `GetPlayers()`.
- `task.spawn` each call in that loop, so one slow handler cannot hold up the rest.
- Key anything stored about a player by `player.User.Id`, and show people the `DisplayName`.
