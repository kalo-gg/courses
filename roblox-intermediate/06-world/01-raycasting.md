# Raycasting

*Ask what lies along a line with `workspace:Raycast`, and decide what it may hit with `RaycastParams`.*

Roblox Intermediate · The world · 10 min

---

A blaster needs to know what is under the crosshair. A laser should stop at the first wall. A ledge-grab needs to know whether there is floor in front of the feet. Each one asks the same question: what is along this line?

`workspace:Raycast(origin, direction, params)` answers it. It returns the first part the line meets, as a `RaycastResult`, or `nil` when nothing is there.

## One line, one answer

```luau
local turret = workspace.Turret
local direction = turret.CFrame.LookVector * 300 -- 300 studs ahead

local result = workspace:Raycast(turret.Position, direction)
if result then
	print(result.Instance.Name) -- the part it hit
	print(result.Position)      -- where the line met its surface
	print(result.Normal)        -- the way that surface faces
	print(result.Distance)      -- how far from the origin
else
	print("nothing within 300 studs")
end
```

The direction carries the range: its length is how far the ray looks, up to 15,000 studs. `LookVector` is one stud long, so it is multiplied by 300.

Without params, a ray stops at the first part it meets, and in a shooter that is often your own arm. A `RaycastParams` says which parts count:

- `ExcludeInstances` lists instances the ray passes through, with all their descendants.
- `IncludeInstances` lists the only instances it may hit. `nil` allows everything, but `{}` allows nothing.

An instance in both lists is excluded. Make the params once and reuse them.

```luau
local params = RaycastParams.new()
params.ExcludeInstances = { player.Character } -- never hit yourself
local result = workspace:Raycast(origin, direction, params)

local floorsOnly = RaycastParams.new()
floorsOnly.IncludeInstances = { workspace.Map.Floors } -- floors and nothing else
```

Each list covers descendants, so `{ player.Character }` skips every limb and accessory in the character.

| RaycastParams property | What it does |
| --- | --- |
| `ExcludeInstances` | These instances and their descendants are never hit |
| `IncludeInstances` | Only these instances and their descendants can be hit |
| `CollisionGroup` | Skips parts in groups set not to collide with this one |
| `RespectCanCollide` | Uses each part's `CanCollide` instead of its `CanQuery` |
| `IgnoreWater` | Passes through Terrain water |

`OverlapParams`, for the next lesson's volume queries, takes the same two lists.

> **Recognising older code**
>
> Older scripts filter with a pair, and the raycasting guide still shows it: `params.FilterType = Enum.RaycastFilterType.Exclude` (once named `Blacklist`) plus `params.FilterDescendantsInstances = { character }`. The `RaycastParams` docs call that pair superseded by `ExcludeInstances` and `IncludeInstances`. Older still is `workspace:FindPartOnRay(ray, ignore)`, which is deprecated.

## Aiming from the camera

Module 5 turned the mouse into a ray with `camera:ViewportPointToRay` and `UserInputService:GetMouseLocation()`. That ray starts at the camera and is one stud long. Scale its direction to a range, exclude the player's own character, and the hit's `Position` is the point in the world under the mouse: what a building tool places at and a blaster aims at.

**Key points**

- `workspace:Raycast(origin, direction, params)` returns the first hit as a `RaycastResult`, or `nil`.
- The direction's length is the range: multiply a unit direction by how far you want to look.
- `ExcludeInstances` and `IncludeInstances` each cover descendants; `IncludeInstances = {}` allows nothing.
- `FilterType` with `FilterDescendantsInstances` is the superseded way to filter.
