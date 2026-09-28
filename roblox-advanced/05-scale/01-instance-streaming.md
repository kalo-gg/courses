# Instance streaming

*Tune streaming for the whole place, keep one model loaded for the player who needs it, and fetch an area before moving a player there.*

Roblox Advanced · Scale · 12 min

---

A tycoon's HUD has a "Go home" arrow, and its client script reads `workspace.Plots.Plot4.Base.Position` to aim it. On the far side of the map it errors: the plot has streamed out of that player's client.

Roblox Intermediate gave the client-side answer: wait for what you cannot do without, and skip what you can. This lesson works on the other side: the settings that shape streaming, and the models that opt out.

## Settings for the whole place

Streaming is tuned on Workspace, in Studio: scripts cannot set these properties. The streaming guide recommends these values:

| Workspace property | Recommended | What it does |
| --- | --- | --- |
| `StreamingMinRadius` | `64`, the default | Content this close to the player streams in first and never streams out. |
| `StreamingTargetRadius` | `1024`, the default | The farthest content streams in. Keep it above the minimum: the gap is a buffer. |
| `StreamOutBehavior` | `Opportunistic` | Removes content beyond the target radius even when memory is not low. |
| `ModelStreamingBehavior` | `Improved` | Sends a model with parts only when one of its parts streams in. |
| `StreamingIntegrityMode` | `PauseOutsideLoadedArea` | Pauses a player who moves into an area that has not streamed in yet. |

## One model at a time

Each Model also has a `ModelStreamingMode`. The default behaves as `Nonatomic`: the model's parts stream in and out on their own. The other modes trade memory for certainty, so use them only where a client script needs the model. The guide reserves `Persistent` for rare cases and says it is not a way around streaming.

| `ModelStreamingMode` | Streams | Use it for |
| --- | --- | --- |
| `Default`, `Nonatomic` | Each part on its own | Most models |
| `Atomic` | The whole model in and out together | A model a client script reads as a unit |
| `Persistent` | Sent soon after joining, never streamed out | A few parts every client must always reach |
| `PersistentPerPlayer` | Persistent for the players added with `AddPersistentPlayer`, atomic for everyone else | Something one player owns |

Set the mode in Studio or a server Script, never a LocalScript. Client code waits for `Workspace.PersistentLoaded` before it reads persistent models.

**ServerScriptService/Plots (Script)**

```luau
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local plot = Workspace.Plots.Plot4

-- Atomic for everyone, persistent for the players added below
plot.ModelStreamingMode = Enum.ModelStreamingMode.PersistentPerPlayer

Players.PlayerAdded:Connect(function(player)
	if player.User.Id == plot:GetAttribute("OwnerId") then
		plot:AddPersistentPlayer(player)
	end
end)
```

The owner's client gets Plot4 soon after joining and keeps it; everyone else streams it as one unit. `AddPersistentPlayer` has no effect unless the mode is `PersistentPerPlayer`, and `RemovePersistentPlayer` undoes it.

Before the server moves a player somewhere far, `player:RequestStreamAroundAsync(position)` asks for that area first. It yields, its effect is temporary, and nothing is guaranteed: client memory and the network decide what arrives. With `PauseOutsideLoadedArea` set, a player who still reaches an unloaded area is paused until it loads.

**Server: fetch the area, then move the player**

```luau
local function moveTo(player: Player, destination: Vector3)
	-- Ask for the area first; this yields
	player:RequestStreamAroundAsync(destination)

	local character = player.Character
	if character then
		character:PivotTo(CFrame.new(destination))
	end
end
```

An optional second argument is a timeout in seconds. Without one the request never times out, though the engine drops it if the client runs low on memory.

> **In the sandbox**
>
> Nothing streams here beyond Roblox Intermediate's hide-and-show stand-in. `ModelStreamingMode` is a plain property and `AddPersistentPlayer` is recorded, so the exercise checks your configuration, not what a client receives.

**Key points**

- Streaming is tuned on Workspace in Studio. The guide recommends `PauseOutsideLoadedArea`, `Opportunistic` stream-out and `Improved` model streaming, with the default radii.
- Models default to `Nonatomic`. `Atomic` streams a model as a unit, and `Persistent` sends it once and keeps it, for rare cases only.
- `PersistentPerPlayer` with `AddPersistentPlayer` keeps a model loaded for chosen players only. Set the mode in Studio or a server Script.
- `RequestStreamAroundAsync` fetches an area before the server moves a player there. It helps, but guarantees nothing.
- Client code must cope with a partial, changing world. A check that needs the whole world belongs on the server.
