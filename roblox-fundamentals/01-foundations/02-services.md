# Services

*game:GetService finds a service by its class, and the container you choose decides who can see what is inside it.*

Roblox Fundamentals · Foundations · 9 min

---

A script that greets players needs the Players service. `game.Players` finds it, because dot indexing looks for a child with that name. That only works while the name matches. RunService's `Name` is `"Run Service"`, with a space, and a service that has not been created yet is not a child at all.

`game:GetService` asks by class instead, and it creates the service if it is missing.

## `GetService`

```luau
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

print(RunService.Name)      --> Run Service
print(RunService.ClassName) --> RunService
print(game:GetService("Workspace") == workspace) --> true
```

`GetService` takes a class name, and some services can be created no other way. `workspace` is a global for the same object as `game:GetService("Workspace")`. You will also see `game.Workspace` and `game.RunService`: `DataModel` has properties for those two services, but not for the rest.

**The first lines of a server Script**

```luau
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")

-- then require modules, write local functions and connect events
```

The docs' services guide calls this the first step of the most common pattern in Roblox code. Get each service once, at the top of the script, in a variable named after it.

## The containers

The services you will open most are **containers**, which hold your instances. The container you pick decides two things. Does a script inside it run? And does it **replicate**, which means every player's device gets a copy?

| Service | What goes in it | Players get a copy? |
| --- | --- | --- |
| Workspace | The 3D world: parts, models and terrain | Yes |
| Players | One Player object for each connected player | Yes |
| ReplicatedStorage | Modules and assets that both sides use | Yes |
| ReplicatedFirst | The minimum a loading screen needs, sent before anything else | Yes |
| ServerScriptService | The server's Script and its ModuleScripts | No |
| ServerStorage | Templates to clone, and maps that are not in use yet | No |
| StarterGui | UI that is copied into each player's PlayerGui | Its contents, per player |
| StarterPlayer | Player defaults, and StarterPlayerScripts | Its contents, per player |
| Lighting | The place's global lighting | Yes |

Only the two Server containers stay on the server. Server code and anything a player must not read belong there. Scripts in ServerStorage do not run, but other scripts can still require modules stored there.

**Key points**

- `game:GetService("X")` finds a service by class and creates it if needed. Dot indexing only matches Names.
- Get each service once, at the top of the script, in a variable named after it.
- `workspace` is a global that holds the same object as `game:GetService("Workspace")`.
- ServerScriptService and ServerStorage stay on the server. Workspace, ReplicatedStorage, ReplicatedFirst and Lighting replicate to every player.
