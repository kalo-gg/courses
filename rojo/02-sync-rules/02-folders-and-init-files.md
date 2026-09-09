# Folders and init files

*A directory is a Folder, unless an init file turns the directory itself into a script with the other files as children.*

Rojo · Sync Rules · 9 min

---

A directory becomes a `Folder` with the same name, and the files inside become its children. That covers most organization. But scripts can have children too (a ModuleScript with helper modules under it), and a file cannot contain other files. Rojo's answer is the **init file**: put `init.luau` inside a directory and the directory *becomes* that ModuleScript, with its other files as the script's children.

```text
src/shared/Inventory/
├── init.luau        → the directory becomes ModuleScript "Inventory"
├── Slots.luau       → ModuleScript "Slots", a child of Inventory
└── Types.luau       → ModuleScript "Types", a child of Inventory
```

Inside `init.luau`, `script.Slots` reaches the child, exactly as it would in Studio.

Inside an init file, `script` *is* the directory's instance, so the other files are its children exactly as they would be in Studio. That is the whole reason the rule exists: a ModuleScript that keeps its helpers under itself, or a `Script` whose sub-modules only it can see, needs a directory to hold the files and an init file to be the parent.

**src/shared/Inventory/init.luau**

```luau
-- this file IS ReplicatedStorage.Shared.Inventory
local Slots = require(script.Slots)   -- Slots.luau, a child of Inventory
local Types = require(script.Types)

local Inventory = {}
function Inventory.new()
	return { slots = Slots.new(8) }
end
return Inventory
```

## Which init file wins

| Init file | The directory becomes |
| --- | --- |
| `default.project.json` | That project's tree — checked before any init file |
| `init.luau` | `ModuleScript` |
| `init.server.luau` | `Script` |
| `init.client.luau` | `LocalScript` |
| `init.plugin.luau` | `Script` with RunContext Plugin |
| `init.csv` | `LocalizationTable` |
| `init.meta.json` with `className` | Any class (next lesson) |

The table is a priority list. A `default.project.json` in the directory is checked before anything else and replaces the whole directory with that project's tree. After that, the first init file present wins and the rest are ignored. Only one instance can stand for a directory, so there is no way to be both a ModuleScript and a Script — pick one, and give the other its own directory.

> **One init file per directory**
>
> A directory can only be one instance, so if it holds both `init.luau` and `init.server.luau`, Rojo takes the first in its priority order (`init.luau`, then `init.server.luau`, `init.client.luau`, `init.plugin.luau`, `init.csv`) and **ignores the rest entirely** — they produce no instance, though Rojo still watches them for changes. A `default.project.json` in the directory outranks all of them. Watch sibling names too: `Inventory.luau` next to `Inventory/` gives two instances called `Inventory`. Rojo does not complain, but `script.Inventory` will only ever find one of them.

**Key points**

- A directory is a Folder; its files are the Folder's children.
- An init file (`init.luau`, `init.server.luau`, `init.client.luau`, `init.plugin.luau`, `init.csv`) makes the directory that instance, with the other files as children.
- One init file per directory (the first in priority order wins; the others produce nothing), and a `default.project.json` beats them all; a file and a directory with the same name give two separate instances.
