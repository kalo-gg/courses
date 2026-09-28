# TweenService

*Give TweenService a goal and a TweenInfo, and it moves the properties there along an easing curve.*

Roblox Fundamentals · Motion · 10 min

---

A door should slide up over half a second, slowing as it arrives. With a frame loop you would track the elapsed time, work out the fraction done, bend it into a curve, set the CFrame and disconnect at the end.

For any change with a start, an end and a duration, **TweenService** does all of that. Give it the instance, the goal values and a `TweenInfo` for the timing, and it moves the properties there frame by frame on its own.

## Goal, info, play

**A sliding door**

```luau
local TweenService = game:GetService("TweenService")

local door = workspace.Door
local info = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local open = TweenService:Create(door, info, {
	CFrame = door.CFrame * CFrame.new(0, 8, 0),
})

open:Play()
open.Completed:Wait() -- yields until the tween ends
print("open")
```

The goal table maps property names to end values, and every property in it moves at once. `Create` only builds the tween: nothing moves until `Play`.

`TweenInfo.new` takes up to six values in order, all optional: the time in seconds (1), the easing style (`Quad`), the easing direction (`Out`), the repeat count (0, or -1 for forever), whether each run reverses (false), and a delay before it starts (0).

The style is the shape of the curve. The direction says which end eases: `Out` starts fast and slows into the goal, and `In` does the opposite. A TweenInfo cannot be changed once made, so build one and reuse it.

## Controlling a running tween

A tween can be stopped two ways, and they differ in what happens next. `Completed` tells you which way it ended.

| Call | What it does |
| --- | --- |
| `tween:Play()` | Starts the tween, or resumes it after `Pause`. Does nothing while it is already playing. |
| `tween:Pause()` | Stops it and keeps its progress. `Completed` does not fire. |
| `tween:Cancel()` | Stops it and resets its progress. The property stays where it had got to. |
| `tween.Completed` | Fires with an `Enum.PlaybackState`: `Completed` at the end, `Cancelled` after `Cancel`. |

**A beacon that pulses until the round ends**

```luau
local pulse = TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
local glow = TweenService:Create(workspace.Beacon, pulse, { Transparency = 0.8 })
glow:Play()

-- when the round ends:
glow:Cancel()
workspace.Beacon.Transparency = 0
```

`-1` repeats forever and `true` plays each run back, so the beacon fades out and in until something stops it. `Cancel` leaves the property wherever it was, so set it back yourself. A newer tween on the same property would also cancel this one.

> **Platforms that carry players**
>
> A tween moves a part by setting its property every frame, as your Heartbeat loop did. That suits doors, pickups and fades. For a platform players stand on, test the ride in Studio with a character before you build a level on it. The constraints lesson shows how to let physics move a platform instead.

**Key points**

- `TweenService:Create(instance, info, goals)` builds a tween; nothing moves until `Play`, and `Play` does not yield.
- `TweenInfo.new(time, style, direction, repeatCount, reverses, delay)`: every argument is optional, and the default curve is Quad Out.
- `Completed` fires with a `PlaybackState` when the tween ends or is cancelled; `Completed:Wait()` yields until then.
- `Pause` keeps the progress; `Cancel` resets it and leaves the property where it was.
- A newer tween on the same property cancels the older one.
