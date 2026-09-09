# What Rojo owns

*$ignoreUnknownInstances, partially versus fully managed places, and globIgnorePaths.*

Rojo · Project Files in Depth · 9 min

---

When the plugin syncs, it compares the tree from the CLI with the instances in Studio. An instance in Studio that the project does not describe is **unknown**. Should Rojo delete it or leave it? `$ignoreUnknownInstances` answers per node: `true` leaves unknowns alone, `false` removes them so Studio matches the files exactly.

The default depends on the node. A node **without** `$path` (a pure node like `Workspace` or a service you only configure) defaults to `true`: Rojo assumes you build things there by hand. A node **with** `$path` gets no default and behaves as `false`, which means the plugin deletes strays under it: your `src/server` folder is the whole truth about `ServerScriptService.Server`.

**Saying it out loud**

```json
"Workspace": {
  "$ignoreUnknownInstances": true,
  "$properties": { "FilteringEnabled": true }
},
"ServerScriptService": {
  "Server": { "$path": "src/server", "$ignoreUnknownInstances": false }
}
```

Both lines restate the defaults. Write them anyway on the nodes where the answer matters to a teammate.

| Workflow | Project file | Studio |
| --- | --- | --- |
| Partially managed | Scripts, config, small models | Map, terrain, hand-placed assets; services keep `$ignoreUnknownInstances: true` |
| Fully managed | Everything, including `.rbxm` exports of builds | Nothing survives that is not in the repo; `rojo build` reproduces the place |

> **Before you flip it to false**
>
> Setting `$ignoreUnknownInstances: false` on `Workspace` while the map lives in Studio deletes the map on the next sync. Use `false` only where the files really are complete, and keep a published version or a backup copy of the place outside the repository before the first sync.

## Skipping files

`globIgnorePaths` lists globs (relative to the project file) that Rojo never reads: test files, editor scratch, generated output. Since 7.7.0 a leading `!` re-includes something an earlier pattern excluded, gitignore style. Files Rojo does not understand (`.md`, `.png`) are skipped anyway; the list is for files it *would* sync.

```json
"globIgnorePaths": [
  "**/*.spec.luau",
  "!src/shared/Sanity.spec.luau"
]
```

**Key points**

- `$ignoreUnknownInstances: true` leaves Studio-made instances alone; `false` deletes anything the files do not describe.
- Pure nodes default to `true`; nodes with `$path` treat unknowns as strays.
- `globIgnorePaths` skips files Rojo would otherwise sync; `!pattern` re-includes.
