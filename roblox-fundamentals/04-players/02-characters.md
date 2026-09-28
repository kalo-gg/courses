# Characters

*A character is a Model rebuilt on every spawn: handle each one as it arrives, and find the player from any part of it.*

Roblox Fundamentals · Players · 11 min

---

A script gives each player a speed boost by setting `player.Character.Humanoid.WalkSpeed` when they join. It errors with `attempt to index nil with 'Humanoid'`: the character has not spawned yet. Fix that, and the boost vanishes the first time the player dies.

Both bugs come from one fact. The character is not the player: it is a Model the engine builds on every spawn.

## CharacterAdded, on every spawn

```luau
local Players = game:GetService("Players")

local function onCharacterAdded(character: Model)
	local humanoid = character:WaitForChild("Humanoid") :: Humanoid
	humanoid.WalkSpeed = 24
end

local function onPlayerAdded(player: Player)
	player.CharacterAdded:Connect(onCharacterAdded)
	if player.Character then
		task.spawn(onCharacterAdded, player.Character)
	end
end

Players.PlayerAdded:Connect(onPlayerAdded)
for _, player in Players:GetPlayers() do
	task.spawn(onPlayerAdded, player)
end
```

The same shape twice: connect for what comes later, then handle what is already there. `player.Character` is `nil` until the first spawn, and each respawn is a new Model, so the boost is set on each one.

A character is a Model named after the player. The docs say the Humanoid and body parts exist on the server when `CharacterAdded` fires, while accessories can take a few seconds more. `WaitForChild` returns at once when the child is already there.

| Inside a character | What it is |
| --- | --- |
| `Humanoid` | Health, movement and death. The next lesson. |
| `HumanoidRootPart` | An invisible part that is the model's `PrimaryPart`. |
| Body parts | On an R15 rig, 15 parts such as `Head`, `UpperTorso` and `LeftFoot`. |
| `Accessory` | A hat or other accessory, holding a part named `Handle`. |

> **Infinite yield possible on …**
>
> If `WaitForChild` is still waiting after 5 seconds, the Output shows a warning such as `Infinite yield possible on 'Workspace.Ada:WaitForChild("Shield")'`. It is not an error: the thread keeps waiting. Check the spelling and the parent, or pass a timeout, `WaitForChild("Shield", 2)`, which returns `nil` when the time runs out.

## From a touched part to its player

**The classic one-liner, and where it breaks**

```luau
pad.Touched:Connect(function(hit)
	local player = Players:GetPlayerFromCharacter(hit.Parent)
	-- a foot: hit.Parent is the character, so this finds the player
	-- a hat's Handle: hit.Parent is the Accessory, so this is nil
end)
```

It works for body parts and silently misses hats, which touch things all the time.

`Touched` hands you a part, and on a character that part may be a foot or a hat's `Handle`. So climb to the nearest Model with `hit:FindFirstAncestorOfClass("Model")`, then ask `Players:GetPlayerFromCharacter(model)`. It returns `nil` for an NPC, or any model that is not a player's character.

Characters appear on a `SpawnLocation`, a Part players spawn on. To send one player somewhere else, set `player.RespawnLocation` to a SpawnLocation in Workspace that is `Neutral` or matches the player's team colour. An obby checkpoint is exactly that.

**Key points**

- `player.Character` is `nil` until the first spawn, and every respawn is a new Model: connect `CharacterAdded`, then handle the current character.
- A character holds a `Humanoid`, a `HumanoidRootPart` (its `PrimaryPart`), the body parts and any accessories.
- `WaitForChild` warns `Infinite yield possible` after 5 seconds; pass a timeout when a child may never come.
- From a touched part: `FindFirstAncestorOfClass("Model")`, then `GetPlayerFromCharacter`, which is `nil` for anything that is not a player.
- `player.RespawnLocation` picks the SpawnLocation a player respawns on.
