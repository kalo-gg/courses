# Structuring client and server

*Shared, Server and Client folders, one module that owns every remote, typed payloads, and when Blink takes over.*

Roblox Intermediate · Shipping multiplayer · 10 min

---

Twelve scripts each call `WaitForChild("Shoot")`, and one spells it `"shoot"` and waits forever. Two server scripts both create `LoadoutChanged`, so clients listen to one copy while the server fires the other. Renaming a remote means searching every file.

One module fixes all three by naming every remote once. Required on the server, it creates them. Required on a client, it waits for them.

## One module owns every remote

| Folder | Holds | Seen by |
| --- | --- | --- |
| `ReplicatedStorage.Shared` | Modules both sides require: `Remotes`, config, shared types | The server and every client |
| `ServerScriptService.Server` | The server entry Script and server-only modules | The server only |
| `ReplicatedStorage.Client` | The client entry Script (`RunContext = Client`) and client modules | Every client, and so every exploiter |

One entry Script per side, as the docs' "Script locations" guide recommends. Anything in ReplicatedStorage can be read by an exploiter, so secrets stay under ServerScriptService.

**ReplicatedStorage.Shared.Remotes**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local IS_SERVER = RunService:IsServer()
local folder = if IS_SERVER then Instance.new("Folder") else ReplicatedStorage:WaitForChild("Remotes")

-- The server creates each remote; a client waits for the server's copy
local function remote(className: string, name: string): Instance
	if IS_SERVER then
		local created = Instance.new(className)
		created.Name = name
		created.Parent = folder
		return created
	end
	return folder:WaitForChild(name)
end

local Remotes = {
	Shoot = remote("RemoteEvent", "Shoot") :: RemoteEvent,
	BlastEffect = remote("UnreliableRemoteEvent", "BlastEffect") :: UnreliableRemoteEvent,
	RequestLoadout = remote("RemoteFunction", "RequestLoadout") :: RemoteFunction,
}

if IS_SERVER then
	folder.Name = "Remotes"
	folder.Parent = ReplicatedStorage
end

return Remotes
```

Each field is cast to its class, so `Remotes.Shoot:FireServer` autocompletes and a typo like `Remotes.Shot` is a type error. Every script now writes `require(ReplicatedStorage.Shared.Remotes)` and never a remote's name.

## Typed payloads

Payloads can be typed the same way (Luau Intermediate covers the syntax). Give each request a function with typed parameters, and a client that sends the wrong thing is an editor error.

The server cannot be protected like that. Types are erased before code runs, and an exploiter never calls your function, so the handler takes `unknown` and checks it.

```luau
-- Client: the payload is checked where you send it
local function shoot(hitPart: BasePart, hitPosition: Vector3)
	Remotes.Shoot:FireServer(hitPart, hitPosition)
end

-- Server: the payload is unknown until the handler checks it
Remotes.Shoot.OnServerEvent:Connect(function(player: Player, hitPart: unknown, hitPosition: unknown)
	if typeof(hitPart) ~= "Instance" or typeof(hitPosition) ~= "Vector3" then
		return
	end
	-- ...
end)
```

Annotating the handler's parameters as `BasePart` and `Vector3` would compile, and would lie: the engine hands over whatever the client sent.

When remotes multiply, **Blink** writes this module for you. One schema file lists every event and its payload types, and the compiler writes a server module and a client module that pack payloads into buffers and reject malformed reads. The Blink course covers it. For a handful of remotes, the hand-written module is enough.

**Key points**

- Shared modules in `ReplicatedStorage.Shared`, server code under `ServerScriptService`, client code in `ReplicatedStorage.Client`.
- One `Remotes` module names every remote once: the server creates them, clients wait for them.
- Scripts require the module and never spell a remote's name, so a typo is a type error, not a silent wait.
- Typed send functions check your own client; the server's handler still takes `unknown` and validates.
- Blink generates this layer from one schema file when remotes multiply.
