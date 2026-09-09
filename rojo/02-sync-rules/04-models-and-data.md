# Models and data files

*Describe small trees in .model.json, turn data files into ModuleScripts and values, and drop in saved .rbxm models.*

Rojo · Sync Rules · 10 min

---

Not everything is a script. Rojo has a text format for small instance trees, turns data files into modules, and accepts models exported from Studio for the rest.

**src/assets/Sword.model.json**

```json
{
  "className": "Tool",
  "properties": { "CanBeDropped": false },
  "children": [
    {
      "name": "Handle",
      "className": "Part",
      "properties": { "Size": [1, 4, 1] }
    }
  ]
}
```

The top-level instance is named after the file (`Sword`); children carry their own `name`. Every node may have `properties`, `attributes` and `children`.

## Data files

| File | Instance | What Rojo does |
| --- | --- | --- |
| `Name.model.json` | Whatever `className` says | Builds the described tree; comments allowed (`.model.jsonc` too) |
| `Name.json`, `.jsonc` | `ModuleScript` | Source becomes `return { … }` with the data as a Luau table |
| `Name.toml`, `.yaml` | `ModuleScript` | Same idea; TOML dates become strings, YAML since 7.6.0 |
| `Name.txt` | `StringValue` | The text is the `Value` |
| `Name.csv` | `LocalizationTable` | Roblox's localization CSV columns |
| `Name.rbxm`, `.rbxmx` | The saved model | Inserted as-is; edit it in Studio and re-export |
| `Name.project.json` | That project's tree | Composing trees |
| anything else | nothing | Ignored, so a README next to your code is fine |

JSON, TOML and YAML files all become ModuleScripts whose `Source` is `return { … }` with the data written out as a Luau table, so any script can `require` them and read fields directly. `.jsonc` (comments and trailing commas) has been accepted since 7.7.0 and `.yml`/`.yaml` since 7.6.0; TOML datetimes come through as strings. Text files become a `StringValue` and CSV files, in Roblox's localization column layout, a `LocalizationTable`.

**Config.json → ModuleScript "Config"**

```luau
-- src/shared/Config.json: { "maxPlayers": 12, "roundLength": 90 }
-- syncs as a ModuleScript whose Source is:
return {
	maxPlayers = 12,
	roundLength = 90,
}
-- so a script reads require(ReplicatedStorage.Shared.Config).maxPlayers
```

> **Config as JSON**
>
> A `Config.json` holding `{ "maxPlayers": 12 }` becomes a ModuleScript whose source is `return { maxPlayers = 12 }`. Scripts `require` it like any module, and designers edit plain JSON without touching Luau.

For anything you cannot describe by hand — a mesh-heavy prop, a rig, a hand-built room — export it from Studio as `.rbxm` (binary) or `.rbxmx` (XML) and drop the file in. Rojo inserts it as saved. It carries its own properties, so a `.meta.json` beside it is read by nobody; edit the model in Studio and export again. Files that match no rule at all, such as a `README.md` or a `.png`, are simply ignored.

**Key points**

- `Name.model.json` describes an instance tree in text: `className`, `properties`, `attributes`, `children` with `name`s.
- `.json`, `.toml` and `.yaml` become ModuleScripts that return the data; `.txt` a StringValue; `.csv` a LocalizationTable.
- `.rbxm`/`.rbxmx` models are inserted as exported; unknown file types are ignored.
