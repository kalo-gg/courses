# Scripts and where they run

*Script, LocalScript and ModuleScript, RunContext, and the recommended layout: one entry point for each side and modules for the rest.*

Roblox Fundamentals · Foundations · 11 min

---

Some code must run on the server, where it decides who scored and no player can change it. Other code must run on each player's device: the camera, the HUD, the keys. A script in the wrong place runs on the wrong side, or not at all. Three things decide which: its class, its `RunContext` and where it sits.

## Three classes of script

| Class | Runs on | Starts |
| --- | --- | --- |
| `Script` | The server or a client, as its `RunContext` says | By itself, once it is somewhere it can run |
| `LocalScript` | A client only | By itself, from a player's copy of a Starter container |
| `ModuleScript` | Whichever side requires it | Never by itself: `require` runs it once and returns its value |

`RunContext` is a Script property that you set in the Properties window; code cannot change it. Roblox's Script locations guide recommends:

- one Script with `RunContext` **Server** in ServerScriptService, the server's entry point
- one Script with `RunContext` **Client** in ReplicatedStorage, the client's entry point
- ModuleScripts for as much code as possible, required by those two

**ServerScriptService/Round (ModuleScript)**

```luau
local Round = {}

function Round.start()
	print("Round started")
end

return Round
```

**ServerScriptService/Main (Script, RunContext Server)**

```luau
local Lobby = require(script.Parent.Lobby)
local Round = require(script.Parent.Round)

Lobby.start()
Round.start()
```

`script` is the Script that is running, so `script.Parent.Round` is the module next to it. In the guide's pattern every `require` comes before any `start`, so all the modules have loaded before any of them runs.

## `require`, and a look at the client

`require` takes the ModuleScript instance, runs it the first time, and returns what the module returned. Later requires on the same side get the same value back. Luau Fundamentals covered writing modules; the only new part is passing an instance.

The **client** is the Roblox app on each player's device. It runs the client entry point against its copy of what replicates. Roblox Intermediate covers input, UI, the camera and messages between the two sides. This course stays on the server.

> **Recognising older code**
>
> A new Script's `RunContext` is `Legacy`. A Legacy Script runs on the server, and only inside a server container such as Workspace or ServerScriptService. Older games put one in every door, reading `script.Parent`. That still works, but the guide advises against scripts inside Workspace parts. Tag the parts instead and handle them all from one module, as Module 3 shows.

**Key points**

- A Script runs by itself, on the side its `RunContext` names. A ModuleScript runs only when required. A LocalScript runs only on a client.
- The recommended layout is one Server Script in ServerScriptService, one Client Script in ReplicatedStorage, and ModuleScripts for the rest.
- `require(script.Parent.X)` runs a module once and returns its value. Later requires on the same side get the same value.
- Use LocalScripts sparingly, and keep scripts out of Workspace parts.
