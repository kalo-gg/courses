# RemoteEvent

*Fire a message from the client to the server and back, with the engine naming who sent it.*

Roblox Intermediate · Remotes · 10 min

---

A player presses a **Claim** button, and a client script adds 50 to their `Coins`. The number goes up on their screen and nowhere else. As Module 1 showed, a client's changes stay on that client, and the server never heard about it.

To change the game, the client has to *ask*, and the server has to decide. A `RemoteEvent` carries that request across, and it can carry messages back.

## One remote, two halves

**main.server.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local greet = Instance.new("RemoteEvent")
greet.Name = "Greet"
greet.Parent = ReplicatedStorage

greet.OnServerEvent:Connect(function(player: Player, mood: string)
	print(player.DisplayName .. " says hi, feeling " .. mood)
end)
```

**main.client.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local greet = ReplicatedStorage:WaitForChild("Greet")
greet:FireServer("happy")
```

`FireServer` does not wait for the server. The server's handler receives `player` first and `"happy"` second: the engine puts the sender in front of whatever the client sent.

That first parameter is the one argument the server can trust. The engine fills it in from the connection the message came in on, so a client cannot claim to be someone else. Everything after it is whatever the client chose to send, and Module 3 is about checking it.

## Which way the message goes

| Direction | The sender calls | The receiver listens to |
| --- | --- | --- |
| Client to server | `remote:FireServer(...)` | `remote.OnServerEvent`, with `(player, ...)` |
| Server to one client | `remote:FireClient(player, ...)` | `remote.OnClientEvent`, with `(...)` |
| Server to every client | `remote:FireAllClients(...)` | `remote.OnClientEvent`, with `(...)` |
| Client to another client | No method | Fire the server, and let it pass the message on |

`OnClientEvent` gets only the arguments. On a client, a message can only have come from the server.

Create every remote on the server, in ReplicatedStorage, where both sides can see it. A remote a client creates exists only on that client, so the server never hears it. The client finds the server's remote with `WaitForChild`, because it may not have replicated yet.

**Key points**

- Create remotes on the server, in ReplicatedStorage; the client finds them with `WaitForChild`.
- `FireServer(...)` raises `OnServerEvent(player, ...)`: the engine adds `player`, and a client cannot forge it.
- `FireClient(player, ...)` reaches one client and `FireAllClients(...)` reaches every client; both raise `OnClientEvent(...)`.
- Clients never message each other directly: the server relays, and decides.
