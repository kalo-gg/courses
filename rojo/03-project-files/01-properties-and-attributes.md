# Properties and attributes

*Implicit values Rojo can type by itself, explicit { "Type": value } wrappers, and $attributes.*

Rojo · Project Files in Depth · 12 min

---

`$properties` (and `properties` in meta and model files) takes a map of property names to values. Rojo ships a database of every Roblox class and property type, so for most properties you write the **implicit** form: plain JSON that Rojo converts to the property's type.

```json
"Baseplate": {
  "$className": "Part",
  "$properties": {
    "Anchored": true,
    "Material": "Grass",
    "Size": [512, 20, 512],
    "Color": [0.39, 0.37, 0.39],
    "CFrame": [0, -10, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1]
  }
}
```

Vector3 is three numbers, Color3 three floats from 0 to 1, CFrame twelve numbers (position, then the rotation matrix), an Enum its item name.

Some types have no natural JSON shape, and sometimes a property is newer than Rojo's database. For those, use the **explicit** form: an object with a single key naming the type. `{ "UDim2": [[1, 0], [0, 48]] }`, `{ "Color3uint8": [163, 162, 165] }`, `{ "BrickColor": 194 }`. Explicit values are not checked against the property's declared type, and they are the only way to set a property Rojo's database does not know yet.

| Type | Implicit | Explicit |
| --- | --- | --- |
| Bool, String, numbers | `true`, `"text"`, `42` | `{ "Bool": true }` |
| Enum | `"Voxel"` | `{ "Enum": 5 }` |
| Vector3 / Color3 | `[1, 2, 3]` | `{ "Vector3": [1, 2, 3] }` |
| UDim2 | none | `{ "UDim2": [[1, 0], [0, 48]] }` |
| Color3uint8 | none | `{ "Color3uint8": [255, 0, 0] }` |
| Font | `{ "family": "rbxasset://fonts/families/Inconsolata.json", "weight": "Regular", "style": "Normal" }` | same, wrapped in `Font` |

## Attributes and tags

Attributes are a property too, `Attributes`, but Rojo has no class database to consult for them: an attribute can be any type, so it resolves each value on its own. Booleans, numbers and strings may be written plainly (`"Speed": 16` is a `Float64`); **everything else needs the explicit wrapper**, including an `Int32`, a `Vector3`, a `Color3` or an enum. When in doubt, write the type: `"Attributes": { "Speed": { "Float64": 16 } }`. `$attributes` (or `attributes` in meta files) is a shorthand for the same thing, and it **replaces** whatever `Attributes` the path or model already provided. `Tags` is a plain array of strings.

**Attributes beside properties**

```json
"Spawn": {
  "$className": "SpawnLocation",
  "$properties": { "Anchored": true, "Duration": 0 },
  "$attributes": {
    "Team": "Red",
    "Capacity": { "Int32": 4 },
    "Offset": { "Vector3": [0, 3, 0] }
  }
}
```

`"Red"` is typed on its own as a string; the integer and the vector need their wrappers, because nothing tells Rojo what type an attribute is meant to be.

> **Two properties you cannot set**
>
> `Name` and `Parent` are decided by the project file's structure, not by `$properties`. Write either one and Rojo logs “Property 'Name' cannot be set manually, ignoring”, skips it and carries on — so the node stays named after its key and nothing tells you at a glance. Rename the key (or the file) instead.

**Key points**

- Implicit values are plain JSON typed from Rojo's property database; explicit values wrap the type name and skip validation.
- UDim, UDim2, BrickColor, NumberRange and sequences are explicit only.
- Attributes take a plain bool, number or string; anything else (an `Int32`, a `Vector3`, an enum) needs the explicit wrapper.
- `$attributes` replaces the `Attributes` property rather than merging into it; `Tags` is an array of strings.
- `$properties` cannot set `Name` or `Parent`: Rojo logs “Property 'Name' cannot be set manually, ignoring” and moves on. The key in the project file is the name.
