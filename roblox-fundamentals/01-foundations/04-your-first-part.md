# Your first part

*Instance.new, four properties, and Parent set last: an anchored platform in the world.*

Roblox Fundamentals · Foundations · 10 min

---

The obby's first stage is a platform floating ten studs up. You could drag one out in Studio, but a game that builds levels or spawns rewards has to make its parts in code.

`Instance.new(className)` creates an instance of any class that can be created. The new part has no `Parent`, so it is in no tree yet and nobody can see it.

## A platform in code

```luau
local Workspace = game:GetService("Workspace")

local part = Instance.new("Part")
part.Name = "Platform"
part.Size = Vector3.new(8, 1, 8)
part.Position = Vector3.new(0, 10, 0)
part.Anchored = true
part.Color = Color3.fromRGB(0, 170, 255)
part.Parent = Workspace
```

Every line before the last one changes a part that is not in any tree. The last line puts the part into the world, finished.

`Size` and `Position` are **Vector3** values: x, y and z, in studs. `Position` is the centre of the part. `Color3.fromRGB` takes red, green and blue values from 0 to 255. `Anchored = true` keeps the part where you put it, because an unanchored part falls. Module 2 covers property types, and Module 5 covers the vector maths.

| Property | Type | A new Part starts with |
| --- | --- | --- |
| `Size` | Vector3 | 4, 1.2, 2 |
| `Position` | Vector3 | 0, 0, 0 |
| `Anchored` | boolean | `false` |
| `Color` | Color3 | Grey: 163, 162, 165 |
| `Parent` | Instance? | `nil` |

## Why `Parent` goes last

The docs give the reason. Once an instance's `Parent` is set, the engine listens to its property changes for replication, rendering and physics. If you set `Parent` first, every later line is a separate change that the engine must process and send to each player. If you set it last, the part arrives once, complete.

**A staircase: the same pattern in a loop**

```luau
for i = 1, 5 do
	local step = Instance.new("Part")
	step.Name = "Step" .. i
	step.Size = Vector3.new(4, 1, 4)
	step.Position = Vector3.new(i * 6, 5 + i * 2, 0)
	step.Anchored = true
	step.Parent = Workspace
end
```

Each step is finished before it enters the world. `Parent` is still the last line.

> **Recognising older code**
>
> Tutorials often write `Instance.new("Part", workspace)`. The second argument is the parent, so the part is in the world before any property is set. The docs discourage this form. Create the part, set its properties, then set `part.Parent = Workspace` last.

**What you can do now**

- Think of a place as a tree of instances under `game`, and change that tree from a script.
- Get services with `game:GetService`, and pick the container that decides who sees an instance.
- Lay code out as one Server Script in ServerScriptService, with ModuleScripts joined by `require(script.Parent.X)`.
- Build a part with `Instance.new("Part")`, setting `Size`, `Position`, `Anchored` and `Color` first.
- Set `Parent` last, so the part enters the world once and complete.
