# Object and world space

*`cf * offset` measures the offset from the part itself: in front, above or to the right, whichever way it faces.*

Roblox Fundamentals · Space · 11 min

---

A guard should carry a lantern 5 studs in front of it and 2 studs up. Add `Vector3.new(0, 2, -5)` to its position and the lantern goes 5 studs towards the world's `-Z`, which is only in front while the guard happens to face that way.

The offset has to be measured in the guard's own **object space**, where `-Z` is always its front. The fixed axes everything shares are **world space**.

## Multiplication order

```luau
local guard = workspace.Guard

-- object space: 2 up and 5 in front of the guard, whichever way it faces
lantern.CFrame = guard.CFrame * CFrame.new(0, 2, -5)

-- world space: 2 up and 5 towards world -Z, whichever way the guard faces
lantern.CFrame = guard.CFrame + Vector3.new(0, 2, -5)
```

`a * b` applies `b` in `a`'s space. Read the first line as: start at the guard, then move 2 up and 5 forward as the guard sees it.

The same rule applies to turning. `part.CFrame * CFrame.Angles(0, math.rad(45), 0)` turns the part on the spot, around its own up axis. Put the rotation on the left and the part swings around the world's origin instead, as if on the end of an arm. When in doubt, put the part's CFrame first.

## Measuring from a part

Going the other way asks where something is *relative to* a part. `ToObjectSpace` turns a world CFrame into one measured from the part, and `PointToObjectSpace` does the same for a point. In object space the signs read directly: negative `Z` is in front, positive `X` is to the right and positive `Y` is above.

`cf:Inverse()` is the CFrame that undoes `cf`, and both conversions are multiplication underneath, as the table shows.

```luau
local root = character.HumanoidRootPart
local seen = guard.CFrame:PointToObjectSpace(root.Position)

if seen.Z < 0 and math.abs(seen.X) < 4 then
	print("straight ahead of the guard")
end

print(guard.CFrame:ToObjectSpace(lantern.CFrame).Position) --> 0, 2, -5
```

`PointToObjectSpace` undoes the guard's move and turn, so the result is the point as the guard sees it. The last line reads back the offset the lantern was placed with.

| Method | Same as | Turns |
| --- | --- | --- |
| `cf:ToWorldSpace(c)` | `cf * c` | A CFrame relative to `cf` into world space |
| `cf:ToObjectSpace(c)` | `cf:Inverse() * c` | A world CFrame into one relative to `cf` |
| `cf:PointToWorldSpace(v)` | `cf * v` | A point relative to `cf` into world space |
| `cf:PointToObjectSpace(v)` | `cf:Inverse() * v` | A world point into one relative to `cf` |

**Key points**

- `cf * offset` applies the offset in `cf`'s own space; swapping the operands changes the answer whenever `cf` is rotated.
- Adding a Vector3 to a CFrame moves it in world space and keeps its rotation.
- `ToObjectSpace` and `PointToObjectSpace` measure from a part: negative Z is in front, positive X to the right.
- `cf:ToObjectSpace(c)` is `cf:Inverse() * c`, and `cf:ToWorldSpace(c)` is `cf * c`.
