# Sourcemaps and editors

*rojo sourcemap tells your language server which file backs which instance; luau-lsp reads it for you.*

Rojo · The Rojo Workflow · 9 min

---

Your editor sees files; Roblox scripts reference instances (`script.Parent.Config`, `ReplicatedStorage.Shared.Util`). A **sourcemap** bridges the two: a JSON tree of `{ name, className, filePaths, children }` that a language server reads to resolve those paths, offer completions and type `require` results. `rojo sourcemap` writes it from the same project file the plugin uses.

```bash
rojo sourcemap --watch -o sourcemap.json         # keep it fresh while editing
rojo sourcemap --include-non-scripts -o sourcemap.json   # also list models and values
rojo sourcemap --absolute                        # absolute file paths, printed to stdout
```

**sourcemap.json (excerpt)**

```json
{
  "name": "arena",
  "className": "DataModel",
  "children": [
    { "name": "ReplicatedStorage", "className": "ReplicatedStorage", "children": [
      { "name": "Shared", "className": "Folder", "children": [
        { "name": "Util", "className": "ModuleScript", "filePaths": ["src/shared/Util.luau"] }
      ] }
    ] }
  ]
}
```

Four keys per node: `name`, `className`, `filePaths`, `children`. Non-script instances appear only with `--include-non-scripts`.

## luau-lsp and the editor

**luau-lsp** (the standard Luau language server, also a VS Code extension) reads `sourcemap.json` automatically and can regenerate it itself by running `rojo sourcemap --watch`. With it in place, `require(ReplicatedStorage.Shared.Util)` resolves to `src/shared/Util.luau`, so types and go-to-definition work across the whole tree. Add `sourcemap.json` to `.gitignore`; `rojo init` has done that for you since 7.6.1.

Two extensions cover the rest of the editor. The **Rojo** VS Code extension adds a menu to start and stop `rojo serve` and to install the Studio plugin, and it reads the project files in the workspace to offer them; it calls the `rojo` on your PATH, which is why Rokit still does the installing. The **luau-lsp** extension runs `rojo sourcemap --watch` for you when `luau-lsp.sourcemap.autogenerate` is on, against the file named by `luau-lsp.sourcemap.rojoProjectFile` (`default.project.json` unless you change it), so the map is regenerated as you type — note that the extension's `luau-lsp.sourcemap.includeNonScripts` defaults to **true**, while the CLI's own default is scripts only.

> **Piping the map**
>
> Leave `-o` off and `rojo sourcemap` prints the JSON to stdout, which is handy for a quick `| jq '.children[].name'` or for a tool that takes the map on its input.

**Key points**

- `rojo sourcemap -o sourcemap.json --watch` writes a JSON tree of names, classes and file paths.
- luau-lsp reads the sourcemap so instance paths resolve to files; keep `sourcemap.json` out of git.
- The Rojo and luau-lsp editor extensions call the CLI you installed; neither replaces Rokit.
