# Code on the client

*The client entry point, where client code can and cannot run, the four RunContext values, and Players.LocalPlayer.*

Roblox Intermediate · Client and server · 10 min

---

Your HUD needs the player's name, and it needs to know when they press a key. None of that exists on the server: it has no screen and no keyboard, and no single player it could call "me". That code has to run on the player's device.

Roblox Fundamentals set up the server's entry point. This lesson writes the client's.

## The client entry point

**ReplicatedStorage/Client/Main (Script, RunContext Client)**

```luau
local Players = game:GetService("Players")

local Hud = require(script.Parent.Hud)
local Controls = require(script.Parent.Controls)

local player = Players.LocalPlayer
print(`{player.Name} is ready`)

Hud.start()
Controls.start()
```

The server's pattern, on the other side: require every module, then start them. `Players.LocalPlayer` is the Player whose device runs this code.

The docs' Script locations guide recommends one Script with `RunContext` set to **Client**, in ReplicatedStorage, as the single client entry point, with ModuleScripts for everything else. ReplicatedStorage reaches every client, so each player's device runs the Script once. An error points at `ReplicatedStorage.Client.Main`, the same place for every player.

## Where client code runs

| Container | Script with RunContext Client | LocalScript |
| --- | --- | --- |
| ReplicatedStorage | Runs: the recommended entry point | Does not run |
| ReplicatedFirst | Runs, before the rest of the game arrives | Runs |
| StarterPlayerScripts, StarterCharacterScripts | Runs twice: the original and each player's copy | Runs from each player's copy |
| StarterGui, StarterPack | Use a LocalScript here | Runs from each player's copy |
| ServerScriptService, ServerStorage | Never reaches a client | Never reaches a client |

The guide says to use LocalScripts sparingly, and only in the four Starter containers. A copy's errors point at `Players.Ada.PlayerScripts.X`, not at the script you wrote.

`RunContext` has four values. **Legacy**, a new Script's default, runs on the server and only inside a server container. **Server** and **Client** name the side outright. **Plugin** is for code inside a Studio plugin. You set it in the Properties window: its write security is `PluginSecurity`, so a game script that assigns it errors.

Only a client has a player of its own, and the services that talk to the device.

**Only a client has these**

```luau
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer -- nil on the server
local playerGui = player:WaitForChild("PlayerGui") -- this player's screen UI (Module 4)
local camera = workspace.CurrentCamera -- the camera this player looks through (Module 5)

UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
	-- keys, clicks and touches from this device (Module 5)
end)

RunService.PreRender:Connect(function(dt)
	-- every frame, before this device draws it
end)
```

`UserInputService` and `PreRender` work only in client code, and on the server `LocalPlayer` is `nil`. None of these lines belongs in a server Script. A module both sides require can ask where it is running with `RunService:IsClient()` or `RunService:IsServer()`.

> **In the sandbox**
>
> Client exercises are named `main.client.luau`: in a Rojo project with `emitLegacyScripts` off, that suffix makes a Script with `RunContext` Client. Your code runs on one player's device. Other players in the scene are there, but they have no client.

**Key points**

- The recommended client entry point is one Script with `RunContext` Client in ReplicatedStorage, requiring ModuleScripts.
- Use LocalScripts sparingly, and only in StarterPlayerScripts, StarterCharacterScripts, StarterGui or StarterPack.
- `RunContext` is Legacy, Server, Client or Plugin. Set it in Studio: scripts cannot write it.
- `Players.LocalPlayer` is the Player on this device, and `nil` on the server.
- Input, the player's camera, PlayerGui and `PreRender` belong to the client.
