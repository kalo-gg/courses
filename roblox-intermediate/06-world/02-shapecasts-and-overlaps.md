# Shapecasts and overlaps

*Gather every part inside a volume, or sweep a whole shape along a path, and turn parts back into characters.*

Roblox Intermediate · The world · 10 min

---

A grenade goes off. Who is hurt? A ray is a line, and a blast is a ball of space: you want everything inside it, not the first thing along one line. A fireball has the opposite problem. It is two studs wide, and a thin ray slips past a pillar edge the fireball would clip.

WorldRoot has a query for each shape. The **overlap** queries return every part inside a volume. The **shapecasts** sweep a shape along a direction and return the first hit, as `Raycast` does.

## Everything inside a volume

```luau
local params = OverlapParams.new()
params.ExcludeInstances = { workspace.Map }

local inBlast = workspace:GetPartBoundsInRadius(blast.Position, 12, params)
local inZone = workspace:GetPartBoundsInBox(zone.CFrame, zone.Size, params)
```

Both return an array of BaseParts, empty when nothing is there. `OverlapParams` has the same two lists as `RaycastParams`, and `MaxParts` caps how many parts come back (0, the default, means no limit).

Both test each part's **bounding box**, not its true shape, which is fast but loose for balls, cylinders and meshes. `GetPartsInPart` checks true shapes when that matters.

The array holds parts, not characters. A character is many parts, so one player turns up many times. Climb from each part to its model and keep a set of the models already counted.

```luau
local hit = {}
for _, part in workspace:GetPartBoundsInRadius(blast.Position, 12) do
	local model = part:FindFirstAncestorOfClass("Model")
	local humanoid = model and model:FindFirstChildOfClass("Humanoid")
	if humanoid and not hit[model] then
		hit[model] = true
		humanoid:TakeDamage(40)
	end
end
```

A hat's `Handle` sits inside an Accessory, and `FindFirstAncestorOfClass` climbs past it to the character. The `hit` set makes each character take damage once.

> **Recognising older code**
>
> `workspace:FindPartsInRegion3(region, ignore)` and its `WithIgnoreList` and `WithWhiteList` forms are deprecated. `GetPartBoundsInBox` replaces them, and its box can be rotated.

## Sweeping a shape

`workspace:Spherecast(position, radius, direction, params)` moves a ball from `position` along `direction` and returns a `RaycastResult` for the first part it touches, or `nil`. The direction's length is the distance again, up to 1,024 studs, and the radius is capped at 256. A shapecast ignores parts the shape already overlaps where it starts.

```luau
local params = RaycastParams.new()
params.ExcludeInstances = { caster }

local result = workspace:Spherecast(hand.Position, 2, aim.Unit * 150, params)
if result then
	explode(result.Position) -- where the ball first touched a surface
end
```

Shapecasts take `RaycastParams`, like `Raycast`. `result.Distance` is how far the ball travelled before it touched something.

| Query | Shape | Returns |
| --- | --- | --- |
| `Raycast(origin, direction, params)` | A line | The first hit, or `nil` |
| `Spherecast(position, radius, direction, params)` | A ball, swept | The first hit, or `nil` |
| `Blockcast(cframe, size, direction, params)` | A box, swept | The first hit, or `nil` |
| `Shapecast(part, direction, params)` | A part's own shape, swept | The first hit, or `nil` |
| `GetPartBoundsInRadius(position, radius, params)` | Bounding boxes meeting a sphere | Every such part |
| `GetPartBoundsInBox(cframe, size, params)` | Bounding boxes meeting a box | Every such part |
| `GetPartsInPart(part, params)` | A part's true shape | Every part it overlaps |

Every one is a method of `workspace`. The casts take `RaycastParams`, and the overlaps take `OverlapParams`.

**Key points**

- Overlap queries return every part whose bounding box meets a sphere or a box; `GetPartsInPart` tests true shapes.
- A character is many parts: map each part to its model and count each model once.
- `Spherecast` sweeps a ball and returns the first hit, like `Raycast`, but ignores parts it starts inside.
- Casts take `RaycastParams`; overlaps take `OverlapParams`, which adds `MaxParts`.
