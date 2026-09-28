# Tools and the backpack

*A Tool moves from the Backpack into the character when equipped; its clicks happen on the client, and the server checks every request that follows.*

Roblox Intermediate · Input · 11 min

---

Every player has a sword in their hotbar, and clicking with it should hit whatever is in front of them. The obvious home for that code is a server Script inside the sword, listening for `Activated`. But the tools guide says tool events fire only in client code, because only the player's device knows when input happens.

So a tool has two halves: the client notices the click, and the server decides what it does.

## A Tool and where it lives

A character holds a tool by its `Handle`, a part named exactly that. A tool with no handle, such as a spell, sets `RequiresHandle` to false; with neither, the tool does not work.

Tools in **StarterPack** are copied into each player's **Backpack**, the hotbar, whenever their character spawns. Equipping moves the tool into the character model, and unequipping moves it back. The tools guide's server Scripts count on the server seeing that move: its collectible tool unanchors itself from a Script's `Equipped` handler. This course follows the guide's pattern: on the server, a tool in `player.Character` is in the player's hand.

**Sword > LocalScript (client)**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local tool = script.Parent
local swingSword = ReplicatedStorage:WaitForChild("SwingSword")

tool.Activated:Connect(function()
	-- play the swing animation here, at once
	swingSword:FireServer()
end)
```

A tool is where the docs' advice to use LocalScripts sparingly makes room for one: it runs once StarterPack's copy lands in the Backpack. `Activated` says nothing about where the player aimed.

## The server half

**Script in ServerScriptService (server)**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local swingSword = Instance.new("RemoteEvent")
swingSword.Name = "SwingSword"
swingSword.Parent = ReplicatedStorage

swingSword.OnServerEvent:Connect(function(player)
	local character = player.Character
	local sword = character and character:FindFirstChild("Sword")
	if sword == nil or not sword:IsA("Tool") then
		return -- not in the player's hand
	end
	-- allowed: the server works out what the swing hits
end)
```

The server never asks the client which tool it used. It looks in the character, where the tools guide's pattern has an equipped tool.

The Trust module applies in full. An exploiter can fire `SwingSword` without the sword, with arguments your LocalScript never sends, and a hundred times a second. So the server checks that the tool is equipped, holds each player to a cooldown, and works out the effect from its own state.

> **Recognising older code**
>
> Older gear puts a server Script in the tool that connects `tool.Activated` directly, with no remote. The tools guide says tool events fire only in client code, so this course follows its layout: a LocalScript for the input, a RemoteEvent, and a server Script that checks each request.

> **A cloned tool does not survive death**
>
> A tool the server clones into a Backpack is gone after the next death, because the new Backpack is filled from StarterPack alone. Give it again on `CharacterAdded` if it should last.

**Key points**

- A Tool needs a part named `Handle`, or `RequiresHandle` set to false.
- StarterPack is copied into each Backpack on spawn; equipping moves the tool into the character.
- The tools guide says tool events fire only in client code: a LocalScript fires a remote, and a server Script acts on it.
- Following the tools guide, the server finds the equipped tool in `player.Character` itself, holds each player to a cooldown, and ignores what the client claims.
