# One game, many machines

*One server holds the real game, each player's device runs a client with its own copy, and every message between them takes time.*

Roblox Intermediate · Client and server · 9 min

---

Three friends join your obby, and four copies of the game are now running: one on a Roblox server and one on each friend's device. Ada presses Space and jumps on her screen at once. Bo sees that jump a moment later, after it has crossed from Ada's device to the server and on to his.

## The server and the clients

**One server, one client per player**

```text
Roblox server        the real game: every instance, ServerScriptService,
│                    ServerStorage and every player's data
├── Ada's client     a copy of what the server sends, plus Ada's
│                    screen, keys and camera
├── Bo's client      the same, for Bo
└── Cy's client      the same, for Cy
```

Clients never talk to each other. Anything one player does reaches the others through the server.

The **server** runs on Roblox's machines and holds the real game: every instance, the Scripts in ServerScriptService and every player's data. The docs call it the ultimate authority for the game's state.

A **client** is the Roblox app on one player's device. It holds a copy of what the server sends, draws it, and reads that player's input. Keeping the copies in step is **replication**.

**ServerScriptService/Main (Script, RunContext Server)**

```luau
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
	print(`{player.Name} joined`)
end)
```

One server runs this once, and its log gets a line for every player who joins.

**ReplicatedStorage/Client/Main (Script, RunContext Client)**

```luau
local Players = game:GetService("Players")

print(`Hello, {Players.LocalPlayer.Name}`)
```

Lesson 3 covers where client code lives. `Players.LocalPlayer` is the Player whose device runs the code.

## Latency, and who decides

Each trip across the network takes time, and more on a weak connection. That delay is **latency**, and it is why the client draws and reacts at once instead of waiting for the server.

The client still cannot decide what happens. A player controls their own device, and an exploiter can change anything their client runs or sends. The docs' security guide gives the client one job: render the world and send the player's input. The server checks each request and applies it. Module 3 covers the checks.

**Key points**

- A game runs on one server and one client per player. Clients never talk to each other directly.
- The server holds the real game. Each client holds a copy of what the server sends it.
- Every message between them takes time (latency), so the client reacts at once to feel responsive.
- The server decides everything that matters. The client renders the world and sends the player's input.
