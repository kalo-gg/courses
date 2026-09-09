# Syncback: adopting a Studio place

*Pull a saved place back into files with rojo syncback, limit what comes back with rules, and adopt an existing game.*

Rojo · Packages, Syncback & CI · 10 min

---

Rojo 7.7.0 added the reverse direction. `rojo syncback --input game.rbxl` reads a saved place (or model), walks your project tree, and writes the instances it finds into the folders the tree maps them to: new scripts become `.luau` files, values become `.txt` or `.model.json`, anything Rojo cannot express as files is saved as `.rbxm`. It is how you adopt an existing game, or pull a builder's Studio edits into the repository.

```bash
rojo syncback --input game.rbxl --list --dry-run   # print every path that would change, write nothing
rojo syncback --input game.rbxl --dry-run    # run everything except writing
rojo syncback --input game.rbxl              # Would write 12 files/folders and remove 1 … Is this okay? (Y/N)
rojo syncback --input game.rbxl -y           # no prompt, for scripts
```

## Rules for what comes back

`syncbackRules` in the project file keeps syncback from dragging in things you do not want on disk: `ignoreTrees` skips whole subtrees by path, `ignorePaths` skips file globs, `ignoreProperties` maps class names to property names to leave out, and `syncCurrentCamera` (default false), `syncUnscriptable` (default true), `ignoreReferents` and `createIgnoreDirPaths` fine-tune the rest.

```json
"syncbackRules": {
  "ignoreTrees": ["Workspace/Terrain", "Workspace/Camera"],
  "ignorePaths": ["**/*.rbxm"],
  "ignoreProperties": { "BasePart": ["Color", "Material"] },
  "syncCurrentCamera": false
}
```

> **What syncback refuses**
>
> Syncback is newer than the forward direction and does not cover every file rule yet. `.json`, `.toml` and `.yaml` data files, an `ignore` rule, plugin scripts, and the four `syncRules` handlers that force a script form (`legacyServerScript`, `legacyClientScript`, `runContextServerScript`, `runContextClientScript`) all stop it with “syncback is not implemented … yet” or “cannot syncback Json middleware”. A project that leans on `syncRules` will need those rules out of the way before its first syncback. `$path` nodes that are optional and missing are skipped with a warning, and duplicate sibling names — legal going forwards — are a hard error going back, because two files cannot share a name.

Adopting a game that has only ever lived in Studio is syncback's main job. Write a partial project first — the code roots, `Workspace` and the other services as pure nodes that keep unknowns — then `rojo syncback --input game.rbxl --dry-run --list` and read the list: everything under a `$path` node is about to become a file. Add `ignoreTrees` for terrain and camera, `ignorePaths` for anything better left as `.rbxm`, run it for real, commit, and only then connect the plugin. The confirmation dialog on that first connect should list nothing to remove, because the files now describe exactly what the place holds.

**Key points**

- `rojo syncback --input file.rbxl` writes a place's instances into the project's folders; `--dry-run` previews (add `--list` to print every path), `-y` skips the prompt.
- `syncbackRules` (`ignoreTrees`, `ignorePaths`, `ignoreProperties`, camera and unscriptable flags) limit what comes back.
- To adopt a Studio-built game: partial project, dry-run syncback, rules, real syncback, commit, then connect.
