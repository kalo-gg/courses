# Planning a repository

*Decide which layer owns each instance before you write a line: files, the project file, or the settings beside it.*

Rojo · Structuring a Real Game · 10 min

---

Every rule so far belongs to one of three layers, and a real repository is planned by deciding, instance by instance, which layer owns it. **Files** decide class and nesting: a suffix, an init file, a meta file, a model file. The **project file** decides placement, pure nodes and properties. The **settings** beside `tree` decide ownership and safety: what is ignored, what is removed, which place may connect, which script form is emitted. When something is hard to express, it is usually sitting in the wrong layer.

**The arena, layer by layer**

```text
files       src/server/Round/init.server.luau   class & nesting: a Script with Scoring under it
            src/gui/Hud/init.meta.json          class: a ScreenGui instead of a Folder
            assets/Sword.model.json             a Tool described as text
project     "Shared": { "$path": …, "Remotes": { "$className": "Folder" } }   placement + a pure node
            "Workspace": { "Baseplate": … }     properties nothing on disk holds
settings    "globIgnorePaths": ["**/*.spec.luau"]      what never syncs
            "$ignoreUnknownInstances"           who owns what Studio adds
            "servePlaceIds", "emitLegacyScripts"       safety and script form
```

## Decide ownership before the first sync

| Decision | Layer | Convention |
| --- | --- | --- |
| Where a script runs | project | One root per service, the folder name says which |
| What class a directory is | files | Init file for scripts, `init.meta.json` for anything else |
| Values scripts read but never write | files | `.json` in `src/shared` |
| Instances no file can describe | project | Pure nodes with `$className` |
| What builders keep | settings | `true` on Workspace and the services you only configure |
| What git owns | settings | `false` on every code root |
| Big builds | none | Stay in Studio, or ship as `.rbxm` |

The first sync into a shared, existing place is where a wrong decision costs something, so make four things explicit first: `servePlaceIds` names the development place and nothing else; `Workspace` and every service you merely configure keep unknown instances; every `$path` root removes them; and a backup of the place exists outside the repository. Then connect, and read the confirmation's remove list as a review of those four lines.

**The settings block, as a checklist**

```json
{
  "name": "arena",
  "servePlaceIds": [1234567890],
  "emitLegacyScripts": false,
  "globIgnorePaths": ["**/*.spec.luau"],
  "tree": { "$className": "DataModel" }
}
```

Three lines beside `tree`, plus `$ignoreUnknownInstances` on the nodes inside it. The final project writes all of them.

The next three lessons add the shapes most repositories grow into: a second place sharing `src/shared`, instances that point at each other through ids, and a library that ships as a model with a test place. Each is another project file or another `$` key; none changes the three layers.

**Key points**

- Files decide class and nesting; the project file decides placement, pure nodes and properties; settings decide ownership and safety.
- Before the first shared sync: `servePlaceIds`, unknowns kept on pure nodes, unknowns removed on code roots, a backup outside git.
- A fresh clone is `rokit install`, `wally install`, `rojo build`, `rojo serve`, then work.
- Several places, ids and libraries are more project files and more `$` keys, not new layers.
