# Attributes and tags

*Attach your own values to any instance with attributes, label instances with tags, and find every tagged one.*

Roblox Fundamentals · Instances · 11 min

---

Some parts in an obby hurt, each by a different amount, and one script has to find them all. Names like `Lava25` would need parsing, and an `IntValue` inside every part adds an instance just to hold one number. Every instance has two better places for this: **attributes**, your own named values, and **tags**, labels a script can search for.

## Attributes

```luau
local lava = workspace.Stage3.Lava

lava:SetAttribute("Damage", 25)
print(lava:GetAttribute("Damage")) --> 25
print(lava:GetAttribute("Speed")) --> nil: never set

lava:SetAttribute("Damage", nil) -- removes the attribute

for name, value in workspace.Stage3.Door:GetAttributes() do
	print(name, value) -- every attribute on the door
end
```

`GetAttribute` returns `nil` for an attribute that was never set, so `lava:GetAttribute("Speed") > 0` stops with `attempt to compare number < nil`. Give it a default: `(lava:GetAttribute("Speed") or 0) > 0`.

An attribute holds one value of a supported type: `string`, `boolean`, `number`, or a datatype such as `Vector3`, `Color3` or `CFrame`. A table or an Instance errors. Names use letters, digits, `_`, `.`, `-` and `/`, up to 100 characters, and cannot start with `RBX`. Attributes are saved with the place, shown in Studio's Properties window, and replicated like properties.

## Tags

```luau
local CollectionService = game:GetService("CollectionService")
local lava = workspace.Stage3.Lava

lava:AddTag("Hazard")
print(lava:HasTag("Hazard")) --> true

for _, hazard in CollectionService:GetTagged("Hazard") do
	print(hazard:GetFullName()) -- every Hazard in the place
end

lava:RemoveTag("Hazard")
```

Adding a tag that is already there does nothing, and removing one that is missing does not error. `GetTagged` returns every tagged instance in the DataModel, in no particular order; one with no parent is left out.

Tags say **which** instances; attributes say **how much**. A builder tags a part `Hazard` and types its `Damage` into the Properties window, and one script handles every hazard without knowing a single name. Module 3 extends that script to parts tagged while the game runs.

> **Recognising older code**
>
> Older guides write `CollectionService:AddTag(part, "Hazard")`, `CollectionService:HasTag(part, …)` and `CollectionService:RemoveTag(part, …)`. They still work and fire the same signals. This course writes the Instance methods: `part:AddTag("Hazard")`.

## Attribute or value object?

|  | Attribute | Value object |
| --- | --- | --- |
| What it is | A named value on the instance itself | A child instance: `IntValue`, `StringValue`, `ObjectValue`… |
| Read and write | `GetAttribute`, `SetAttribute` | `.Value` |
| Can hold an instance | No: `SetAttribute` errors | Yes, in an `ObjectValue` |
| Still the right choice for | Almost everything | The leaderboard's `leaderstats` (Module 4), and references to other instances |

Older tutorials use value objects for everything, because attributes came later. Prefer an attribute unless something needs an instance.

**Key points**

- `SetAttribute` stores your own value on any instance and `nil` removes it; `GetAttribute` returns `nil` when it is unset.
- Attributes hold strings, booleans, numbers and datatypes such as `Vector3`, never tables or instances.
- `AddTag`, `HasTag` and `RemoveTag` are Instance methods; `CollectionService:GetTagged` finds every tagged instance.
- Tags pick out which instances; attributes say how much. Prefer both to parsing names or adding value objects.
