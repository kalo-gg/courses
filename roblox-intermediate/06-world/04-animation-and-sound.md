# Animation and sound

*Play animation tracks through the Animator, and build sound from the Audio API's players, wires and outputs.*

Roblox Intermediate · The world · 11 min

---

A player holds G to cheer. Their character should throw up its arms, and a cheer should play until they let go. None of it changes the game's rules: it is feedback, so it runs on the client of the player pressing G.

It takes two systems. The **Animator** plays animation tracks on a rig. The **Audio API** builds sound from parts: players make a stream, and wires carry it to outputs.

## Animation tracks

```luau
local humanoid = character:WaitForChild("Humanoid")
local animator = humanoid:WaitForChild("Animator")

local cheer = Instance.new("Animation")
cheer.AnimationId = "rbxassetid://…" -- an animation your game may use

local track = animator:LoadAnimation(cheer) -- load once, keep the track
track.Looped = true
track:Play()
task.wait(3)
track:Stop()
```

Every `LoadAnimation` call makes a new track, so load once and keep it. The Animator must be in the Workspace, or the call errors.

Which side plays a track decides who sees it. A track a client plays on **its own character's** Animator replicates to the server and to every other player. Any other rig, an NPC or a prop, must be animated from the server for others to see it.

`Play(fadeTime, weight, speed)` and `Stop(fadeTime)` both fade over 0.1 seconds by default. Calling `Play` on a track that is already playing restarts it.

| Topic | What to know |
| --- | --- |
| Markers | Named events set in the Animation Editor. `track:GetMarkerReachedSignal("Step")` fires each time playback reaches one, with the marker's parameter string. |
| Length | `track.Length` is in seconds at speed 1, and reads 0 until the animation has loaded. |
| Permissions | An animation plays in games its creator owns, yours or your group's. Another game needs access, granted on the asset's Permissions page in the Creator Dashboard; without it the animation does not load, and the Output shows an error. |

## Sound is a graph

An `AudioPlayer` loads an asset and produces a stream, but alone it is silent. A `Wire` carries a stream from its `SourceInstance` to its `TargetInstance`. An `AudioDeviceOutput` is the player's speakers or headphones. For 2D sound, the kind a HUD makes, the docs' recipe is those three objects in SoundService.

```luau
local SoundService = game:GetService("SoundService")

local cheerSound = Instance.new("AudioPlayer")
cheerSound.Asset = "rbxassetid://…" -- an audio asset your game may use
cheerSound.Looping = true
cheerSound.Parent = SoundService

local output = Instance.new("AudioDeviceOutput")
output.Parent = SoundService

local wire = Instance.new("Wire")
wire.SourceInstance = cheerSound
wire.TargetInstance = output
wire.Parent = SoundService

cheerSound:Play()
```

`Stop` leaves `TimePosition` where it stopped, and `Play` carries on from there. Set `TimePosition = 0` before `Play` to start from the top.

| For 3D sound | What it does |
| --- | --- |
| `AudioEmitter` | A virtual speaker. Parented to a part or Attachment, it plays what is wired into it from there, quieter with distance. |
| `AudioListener` | A virtual microphone on a camera or character. It picks up nearby emitters; wire it to an `AudioDeviceOutput` to hear them. |
| `SoundService.DefaultListenerLocation` | Set to `Character` or `Camera` in Studio, it creates the listener, wire and output for each player. |

For 3D sound, wire the AudioPlayer into an emitter instead of straight into the output.

> **Recognising older code**
>
> `humanoid:LoadAnimation(animation)` is deprecated: load through the Animator. Many places still play sound with `Instance.new("Sound")`, a `SoundId` and `sound:Play()`. The docs now discourage `Sound`, `SoundGroup` and `SoundEffect` in favour of the audio objects above.

**Key points**

- Load an Animation through the Animator once, keep the track, and call `Play` and `Stop` on it.
- A client's tracks replicate only from its own character; animate NPCs and props from the server.
- 2D sound is an `AudioPlayer`, a `Wire` and an `AudioDeviceOutput`; 3D sound adds an `AudioEmitter` and an `AudioListener`.
- `Humanoid:LoadAnimation` is deprecated, and `Sound` is discouraged in favour of the audio objects.
