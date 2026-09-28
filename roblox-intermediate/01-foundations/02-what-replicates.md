# What replicates

*The server's changes reach every client. A client's changes stay on that device, apart from a few the engine accepts on purpose.*

Roblox Intermediate · Client and server · 9 min

---

An old tutorial gives the player coins from a LocalScript with `leaderstats.Coins.Value += 100`. It seems to work: the player's own client now reads 100. But the server's copy still says 0, and so does every other player's, so when the shop on the server checks the balance, the sword is refused.

## One way: server to clients

Replication runs from the server to the clients. When server code changes a replicated instance, every client's copy follows. When client code changes its own copy, the change stays on that device. The server never hears of it, and can overwrite the client's value at any time.

The same goes for whole instances. A part a client creates exists only on that client, and a part it destroys is gone only from its own copy.

**main.client.luau (on Ada's device)**

```luau
local Workspace = game:GetService("Workspace")

local door = Workspace:WaitForChild("Door")
door.Transparency = 0.5 -- only Ada's copy fades

local marker = Instance.new("Part")
marker.Anchored = true
marker.Position = door.Position + Vector3.new(0, 6, 0)
marker.Parent = Workspace -- only Ada has this part
```

Local-only changes suit effects that one player should see, such as a marker over their next checkpoint. Lesson 4 explains the `WaitForChild`.

**main.server.luau**

```luau
local Workspace = game:GetService("Workspace")

local door = Workspace.Door
door.Transparency = 0.5 -- every client's copy fades
```

The same change made on the server reaches every player. The server has the whole tree, so it can index `Door` directly.

## The exceptions

Roblox Fundamentals sorted the containers: ServerScriptService and ServerStorage never leave the server, so no client can read your server code or templates, and the replicated containers reach every client. Each player also gets their own PlayerGui, filled from StarterGui.

A few client changes do reach everyone, because the engine gives the client that job on purpose. An exploiter's client can use each of them too.

| What a client does | Does the server accept it? |
| --- | --- |
| Moves its own character, or an unanchored part near its character | Yes. The engine gives that client **network ownership** of those parts, so it simulates their physics and the server takes the result. |
| Plays an animation on its own character's Animator | Yes. An Animator in a player's character replicates tracks that player's client starts. The Animator must have been created by the server. |
| Deletes its own character | No, by default. `Workspace.RejectCharacterDeletions` makes the server ignore it, and its `Default` currently means `Enabled`. |
| Plays a `Sound`, the older audio object | Only when `SoundService.RespectFilteringEnabled` is false. Otherwise the sound plays for that client alone. |

Network ownership is why an exploiter can fly or teleport their own character. Roblox Advanced covers checking movement on the server.

> **Recognising older code**
>
> Tutorials from when `Workspace.FilteringEnabled` could be switched off change the world from a LocalScript and expect every player to see it. The property is deprecated and no longer takes effect: a client's changes stay on that client. Ask the server with a remote instead, as Module 2 shows.

**Key points**

- Replication runs from the server to the clients. A client's changes to its copy stay on that device.
- An instance a client creates exists only on that client, and one it destroys is gone only from its copy.
- ServerScriptService and ServerStorage never reach a client. Each player gets their own PlayerGui.
- The engine accepts a few client changes on purpose: moving its own character and the parts it owns, and animations on its own character's Animator.
