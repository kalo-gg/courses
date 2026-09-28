# Models and pivots

*Move a whole model by its pivot with `PivotTo`, and read where it is with `GetPivot` and `GetBoundingBox`.*

Roblox Fundamentals · Space · 10 min

---

A statue is a Model of three anchored parts: a base, a body and a head. Set the base's `CFrame` and only the base moves, leaving the body and head hanging in the air. Moving each part yourself works until the statue also has to turn, and then every offset needs the last lesson's math.

Every Model and every BasePart has a **pivot**: one CFrame that stands for the whole thing. Move the pivot and every part moves with it, keeping its place relative to the others.

## PivotTo and GetPivot

```luau
local statue = workspace.Statue

-- move it: every part keeps its place in the model
statue:PivotTo(CFrame.new(0, 0, 50))

-- turn it a quarter turn where it stands
statue:PivotTo(statue:GetPivot() * CFrame.Angles(0, math.rad(90), 0))

-- lift it 10 studs
statue:PivotTo(statue:GetPivot() + Vector3.new(0, 10, 0))
```

`GetPivot` reads the pivot and `PivotTo` moves it. The docs name `PivotTo` the primary way to move a model from a script, and parts have both methods too.

Where is the pivot? When a model has a `PrimaryPart`, the pivot follows that part. Without one, it is the model's `WorldPivot` property. Character models use their `HumanoidRootPart` as the primary part. In Studio, the **Edit Pivot** tool moves a model's pivot, for example to the bottom of its base, so that `PivotTo(spot)` stands it on `spot`.

```luau
local centre, size = statue:GetBoundingBox()
print(size) --> 4, 7, 4: the box around every part

-- the pivot on the centre of the pad's top face
statue:PivotTo(pad.CFrame * CFrame.new(0, pad.Size.Y / 2, 0))
```

`GetBoundingBox` returns a CFrame at the centre of a box around the model's parts, and the box's size, oriented like the pivot. Half a part's `Size.Y` above its centre is its top face.

`PivotTo` places anchored parts exactly, as setting `CFrame` does. The model keeps track of each part's offset, so moving it again and again does not let the parts drift apart through rounding.

> **Recognising older code**
>
> Older scripts use `model:SetPrimaryPartCFrame(cf)` and `model:GetPrimaryPartCFrame()`. Both are deprecated: the docs call `PivotTo` a more performant replacement and `GetPivot` a drop-in one. `SetPrimaryPartCFrame` errors when the model has no `PrimaryPart`; `PivotTo` works either way.

**Key points**

- Move a model with `PivotTo` and read where it is with `GetPivot`; parts have both methods too.
- The pivot follows `PrimaryPart` when one is set, and is `WorldPivot` otherwise.
- Setting one part's `CFrame` moves that part and what is welded to it, not the whole model.
- `GetBoundingBox` returns the centre and size of a box around the model's parts.
- `SetPrimaryPartCFrame` and `GetPrimaryPartCFrame` are deprecated; `PivotTo` and `GetPivot` replace them.
