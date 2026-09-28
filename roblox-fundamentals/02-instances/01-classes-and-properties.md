# Classes and properties

*Every instance has a class: `IsA` asks about the whole family, `ClassName` only the exact class, and each property holds one type.*

Roblox Fundamentals · Instances · 9 min

---

A folder in Workspace holds a level: two blocks, a wedge, a rock mesh, a spawn pad and a tree model. To anchor every part, you loop over the children and test `child.ClassName == "Part"`. Only the two blocks get anchored. The wedge is a `WedgePart`, the rock a `MeshPart`, the pad a `SpawnLocation`, and the tree a `Model`, which has no `Anchored` property at all.

Every instance is made from a **class**, and the class decides which properties it has.

## Name, ClassName and IsA

```luau
local start = workspace.Level.Start -- a SpawnLocation

print(start.Name) --> Start
print(start.ClassName) --> SpawnLocation
print(start:IsA("SpawnLocation")) --> true
print(start:IsA("Part")) --> true: a SpawnLocation is a kind of Part
print(start:IsA("BasePart")) --> true
print(start:IsA("Model")) --> false
```

`Name` is a label anyone can change, and many instances share one. `ClassName` is fixed when the instance is created. `IsA` is true for the class itself and for every class above it.

Classes form a tree, and every instance's class descends from `Instance`. `BasePart` sits above every physical part: `Part`, `WedgePart`, `MeshPart`, `SpawnLocation` and more. It is **abstract**, so `Instance.new("BasePart")` errors, but `IsA("BasePart")` is how you ask *is this a part of any kind?*

Compare `ClassName` only when the exact class matters, such as telling a `SpawnLocation` apart from other parts.

## Properties have types

Every property has one type, and a value of the wrong type, such as a number for `Size`, errors. Some are plain Luau types: `string`, `number` and `boolean`. Others are Roblox **datatypes** such as `Vector3` and `Color3`, built with constructors. The rest are **enums**: fixed lists of named items under the global `Enum`.

| Property | Type | A value |
| --- | --- | --- |
| `Name` | string | `"Floor"` |
| `Anchored` | boolean | `true` |
| `Transparency` | number | `0.5` |
| `Size` | Vector3 | `Vector3.new(8, 1, 8)` |
| `Color` | Color3 | `Color3.fromRGB(255, 170, 0)` |
| `Material` | Enum.Material | `Enum.Material.Neon` |

Each property's type is listed on its class page in the engine reference. Module 5 covers the maths of `Vector3`.

```luau
local part = Instance.new("Part")
part.Size = Vector3.new(4, 1, 4)
part.Color = Color3.fromRGB(255, 170, 0)
part.Material = Enum.Material.Neon
part.Parent = workspace

print(typeof(part.Size)) --> Vector3
print(part.Material == Enum.Material.Neon) --> true

part.Size = 4
--> Unable to assign property Size. Vector3 expected, got number
```

`typeof` names a Roblox datatype. An enum item is a value you compare with `==`, and a misspelled one, such as `Enum.Material.Neno`, errors instead of doing nothing.

**Key points**

- `Name` is a label you choose; `ClassName` is the class an instance was created as, and never changes.
- `IsA` is true for the class and every class above it, so `IsA("BasePart")` matches every kind of part.
- `BasePart` is abstract: you test for it with `IsA`, but you cannot create one.
- Each property has one type: a Luau type, a datatype such as `Vector3`, or an `Enum` item. `part.Size = 4` errors.
