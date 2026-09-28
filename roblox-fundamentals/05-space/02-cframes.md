# CFrame

*A position plus an orientation: build one with `lookAt` or `Angles`, and read which way it faces.*

Roblox Fundamentals · Space · 10 min

---

A turret should turn to face whatever comes near. `Position` cannot do that: it says where a part is, not which way it points.

A part's rotation lives in its **CFrame**, short for coordinate frame, which holds a position *and* an orientation. Setting `part.CFrame` sets both at once.

## Building a CFrame

```luau
local turret = workspace.Turret
local target = workspace.Target

-- at a position, with no rotation
turret.CFrame = CFrame.new(0, 5, 0)

-- at a position, facing a point
turret.CFrame = CFrame.lookAt(turret.Position, target.Position)

-- at a position, turned a quarter turn around the vertical axis
turret.CFrame = CFrame.new(0, 5, 0) * CFrame.Angles(0, math.rad(90), 0)
```

`CFrame.lookAt(at, target)` is the one to reach for when something should face something else. Passing the turret's own position as `at` keeps it where it is.

`CFrame.Angles` takes **radians**, not degrees. A full turn is `2 * math.pi`, and `math.rad` converts: `math.rad(90)` is a quarter turn. Pass a plain `90` and the part turns 90 radians, over 14 full turns, and ends up pointing somewhere that looks random.

## Which way it faces

A CFrame also knows its own axes. `LookVector` points the way it faces, `RightVector` to its right and `UpVector` out of its top, each with length 1. `turret.CFrame.LookVector` is the facing that the last lesson's `Dot` test needed.

Setting `Position` and setting `CFrame` differ in one way that matters: `Position` moves the part and keeps its rotation, while `CFrame` replaces both.

```luau
local look = turret.CFrame.LookVector
local toTarget = (target.Position - turret.Position).Unit
print(look:Dot(toTarget)) --> about 1 after lookAt: facing it exactly

turret.Position = Vector3.new(0, 5, 20) -- moved, still turned the same way
turret.CFrame = CFrame.new(0, 5, 20)    -- moved, and the turn is gone
```

`LookVector` is the frame's negative Z axis, which is why a part with no rotation faces `-Z`.

| You write | You get |
| --- | --- |
| `CFrame.new(x, y, z)`, `CFrame.new(pos)` | A position with no rotation |
| `CFrame.lookAt(at, target)` | A position, facing a point |
| `CFrame.Angles(rx, ry, rz)` | A rotation in radians, at the origin |
| `cf.Position` | Where it is |
| `cf.LookVector`, `cf.RightVector`, `cf.UpVector` | Its forward, right and up directions |

> **Recognising older code**
>
> Tutorials often face a part with `CFrame.new(pos, target)`. The docs say `CFrame.lookAt(pos, target)` replaced that form, which stays only so old code keeps running. Read it as `lookAt`, and write `lookAt`.

**Key points**

- A CFrame is a position plus an orientation; setting `part.CFrame` sets both.
- `CFrame.lookAt(at, target)` faces a point; `CFrame.new(pos, target)` is the older form of it.
- `CFrame.Angles` takes radians, so wrap degrees in `math.rad`.
- `LookVector`, `RightVector` and `UpVector` are the frame's own forward, right and up.
- Setting `Position` keeps the rotation; setting `CFrame` replaces it.
