# Waiting on the client

*The order a client starts in, why it waits for what arrives later, and what streaming holds back.*

Roblox Intermediate · Client and server · 11 min

---

A client Script reads `workspace.Arena.Scoreboard` on its first line. It works when you test beside the arena. A player spawning across the map gets `Scoreboard is not a valid member of Model "Workspace.Arena"`. The scoreboard exists: this client has not received it yet.

## What a client has when it starts

When a player joins, their client receives ReplicatedFirst and runs the client scripts in it. Then the rest of the game arrives and `game.Loaded` fires. Only then do client Scripts in ReplicatedStorage and LocalScripts in PlayerScripts start. After that the character spawns and its scripts run.

So your entry point can index anything that was in ReplicatedStorage from the start. Workspace objects, and anything the server creates later, arrive in no guaranteed order and need `WaitForChild`.

**ReplicatedFirst/Loading (Script, RunContext Client)**

```luau
-- show the loading screen here

if not game:IsLoaded() then
	game.Loaded:Wait()
end

-- hide it: the rest of the game has arrived
```

Only ReplicatedFirst code starts before `Loaded`. Client scripts anywhere else start after it, so they never need this check.

## Streaming

**Instance streaming** is on by default for new places created in Studio. The server then sends each client only the part of Workspace near its character, so a distant part may not exist on that client yet.

The streaming guide's rule: if the script cannot go on without an instance, wait for it, with a timeout if it might never come. If it can, use `FindFirstChild` and skip it when it is `nil`. Roblox Advanced covers it in depth.

**ReplicatedStorage/Client/Main (Script, RunContext Client)**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- In ReplicatedStorage from the start: already here
local Config = require(ReplicatedStorage.Shared.Config)

-- Created by the server when a round starts: wait for it
local round = ReplicatedStorage:WaitForChild("Round")

-- A Workspace part the HUD needs: wait, but at most 5 seconds
local scoreboard = Workspace:WaitForChild("Scoreboard", 5)
if scoreboard == nil then
	warn("Scoreboard has not streamed in")
end

-- Nice to have: use it if it is here, skip it if not
local statue = Workspace:FindFirstChild("Statue")
if statue then
	statue.Transparency = 0.5
end
```

With a timeout, `WaitForChild` returns `nil` instead of waiting forever. Roblox Fundamentals covered the `Infinite yield possible` warning you get without one.

> **In the sandbox**
>
> The exercise holds one part back from your client, the way streaming holds back a distant part. The server still has it, and it does not stream in during the run.

**Key points**

- A client receives ReplicatedFirst first, then the rest of the game (`game.Loaded`), then runs its client scripts. The character comes last.
- A client entry point can index what was in ReplicatedStorage from the start. Anything created later needs `WaitForChild`.
- Only ReplicatedFirst code needs `game:IsLoaded()` and `game.Loaded`.
- Streaming is on by default for new places, so a distant Workspace part may not exist on the client.
- Wait, with a timeout, for what you cannot do without. Look up what you can skip with `FindFirstChild`.
