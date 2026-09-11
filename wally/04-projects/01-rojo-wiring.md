# Wiring it into a place

*Rojo maps the three installed directories into a place, and a package is a project file of its own.*

Wally · Wally in a Real Project · 10 min

---

`wally install` finishes and you have `Packages/`, `ServerPackages/` and `DevPackages/` sitting in your repository. Nothing in any of them knows what a DataModel is. The tree is files on disk, and files on disk do not appear in a place until something puts them there — which for almost every Roblox project means Rojo, and means one more block in `default.project.json`.

**default.project.json**

```json
{
  "name": "dungeon",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Shared": { "$path": "src/shared" },
      "Packages": { "$path": { "optional": "Packages" } }
    },
    "ServerScriptService": {
      "Server": { "$path": "src/server" },
      "ServerPackages": { "$path": { "optional": "ServerPackages" } }
    },
    "ServerStorage": {
      "DevPackages": { "$path": { "optional": "DevPackages" } }
    }
  }
}
```

Three directories, three homes. The node names are yours to choose — but `[place]` will have to agree with them.

The placement follows the realms. `Packages/` holds shared-realm code that both sides run, so it goes somewhere both sides can see: `ReplicatedStorage`. `ServerPackages/` must never reach a client, so it goes under a server-only service. `DevPackages/` is test tooling, and a great many projects leave it out of the production project file entirely and map it only in a `test.project.json`.

Every one of those paths is wrapped as `{ "optional": "…" }`. That is Rojo's way of saying "skip this node if the folder is not there", and it is what lets somebody clone the repository and build before they have run `wally install` even once.

| Directory | Comes from | Usual home |
| --- | --- | --- |
| `Packages/` | `[dependencies]` | `ReplicatedStorage.Packages` |
| `ServerPackages/` | `[server-dependencies]` | `ServerScriptService.ServerPackages` |
| `DevPackages/` | `[dev-dependencies]` | A test project file, or `ServerStorage` |

Wally chooses the directory from the table you wrote it in. You choose where the directory lands.

## A package is a project

That answer is worth sitting with, because it explains a lot of Wally's behaviour downstream. A package is not a folder Rojo interprets; it is a **project Rojo builds**, and its author decided what it becomes. Most of them decide the same thing: one ModuleScript, pointing at the entry point.

**default.project.json, inside a published package**

```json
{
  "name": "signal",
  "tree": {
    "$path": "src/init.luau"
  }
}
```

Six lines, and the reason `require(Packages.Signal)` gives you a module rather than a folder.

> **`publish` rewrites that name**
>
> `wally publish` overwrites the `name` field of the package's own `default.project.json` with the package's short name before it packs the archive. Whatever you called it locally is not what consumers see, so do not build anything on top of that string — and do not be surprised when the copy in `_Index` disagrees with the copy in your working tree.

Put the two halves together and the require path is unremarkable. Your project file said `Packages` lives in `ReplicatedStorage`; Wally wrote a link module named after the alias in your manifest; the link points into `_Index`, and the package's own project file turns that directory into a module. Four indirections, and none of them visible from the call site.

**src/shared/Chest.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Signal = require(ReplicatedStorage.Packages.Signal)

local opened = Signal.new()

return opened
```

`Packages.Signal` is the alias you wrote in `[dependencies]`, not the package's own name. Rename the alias and this line changes.

**Key points**

- Wally chooses the directory; `default.project.json` chooses where in the datamodel it lands.
- `Packages` goes somewhere replicated, `ServerPackages` somewhere server-only, `DevPackages` usually only in a test project.
- `{ "optional": "Packages" }` lets the project build before anyone has run `wally install`.
- A directory containing a `default.project.json` is described by that file — which is how a package becomes one ModuleScript.
- `wally publish` rewrites that file's `name` to the package's short name, so the local name is not what consumers see.
