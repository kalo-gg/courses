# Rojo reference

*Rojo 7.7.0, released July 2, 2026. Checked on September 7, 2026.*

---

## Command line

Every subcommand takes an optional project path (a directory or a `.project.json`) and defaults to the current directory.

### `rojo init`

```luau
rojo init [PATH] [--kind place|model|plugin] [--skip-git]
```

Creates a new project: a `default.project.json`, a `src/` folder with starter scripts, a README, a `.gitignore` and a git repository. `--kind` picks the template (default `place`); `--skip-git` leaves version control to you.

### `rojo serve`

```luau
rojo serve [PROJECT] [--port 34872] [--address 127.0.0.1] [--allowed-hosts host,…]
```

Starts the live-sync server the Studio plugin connects to. The port comes from `--port`, then the project's `servePort`, then 34872. Binding to a non-loopback address prints a warning: the API is unauthenticated, and `--allowed-hosts` adds accepted `Host`/`Origin` values beyond localhost and the bind address, overriding the project's `serveAllowedHosts`.

### `rojo build`

```luau
rojo build [PROJECT] -o FILE.rbxl|.rbxlx|.rbxm|.rbxmx [--watch] | --plugin FILE.rbxm
```

Writes the project to a file. The extension picks the format: `.rbxl`/`.rbxlx` are places (binary/XML) and drop the DataModel wrapper; `.rbxm`/`.rbxmx` are models and include the root instance. `--watch` rebuilds on every change; `--plugin` writes into Studio's local plugins folder instead of `-o`, and takes a bare file name — an absolute path is refused (“plugin flag path cannot be absolute”) and only `.rbxm`/`.rbxmx` are accepted.

### `rojo sourcemap`

```luau
rojo sourcemap [PROJECT] [-o sourcemap.json] [--watch] [--include-non-scripts] [--absolute]
```

Emits a JSON tree of `{ name, className, filePaths, children }` so editors and language servers know which file backs which instance. Omit `-o` to print to stdout. Scripts only unless `--include-non-scripts`; paths are relative unless `--absolute`.

### `rojo syncback`

```luau
rojo syncback [PROJECT] --input FILE.rbxl [--list] [--dry-run] [-y]
```

The reverse direction (7.7.0): reads a place or model file and writes its instances into the project as files, following the project tree and `syncbackRules`. Asks `Would write X files/folders and remove Y … Is this okay?` unless `-y`; `--dry-run` skips writing; add `--list` to print every path that would change. Not every middleware can go backwards: `.json`, `.toml` and `.yaml` data files, `ignore` rules, plugin scripts and the four forced script forms (`legacyServerScript`, `legacyClientScript`, `runContextServerScript`, `runContextClientScript`) all bail, so a project using `syncRules` or plugin scripts will not sync back as it stands.

### `rojo upload`

```luau
rojo upload [PROJECT] --asset_id ID (--api_key KEY --universe_id ID | --cookie COOKIE)
```

Builds and publishes. With `--api_key` Rojo uses the Open Cloud API (places only, `--universe_id` required); otherwise it uses your Studio cookie, found automatically or passed with `--cookie`. Note the underscores in these three flags.

### `rojo plugin`

```luau
rojo plugin install | rojo plugin uninstall
```

Installs the Studio plugin that matches the CLI version into your local plugins folder, or removes it. The plugin is also on the Creator Store as “Rojo”.

### `rojo fmt-project`

```luau
rojo fmt-project [PROJECT]
```

Rewrites a project file in Rojo's canonical formatting. `rojo doc` opens the documentation site.

## Project file

A `*.project.json` (or `.jsonc`, comments and trailing commas allowed since 7.6.1). `default.project.json` is what every command picks up by default.

### `name`

```luau
"name": "my-game"
```

Name of the root instance in a built model. The project you build or serve must have one unless it is a `default.project.json`, which takes its folder's name (7.4.1). A project reached through a `$path` needs none either: it falls back to its file name, so `foo.project.json` becomes an instance called `foo`.

### `tree`

```luau
"tree": { "$className": "DataModel", … }
```

The root instance description. Required. For a place it is a `DataModel`; for a model or library it is usually a `Folder` or `ModuleScript` path.

### `servePort`

```luau
"servePort": 34872
```

Port `rojo serve` listens on when `--port` is not given.

### `servePlaceIds`

```luau
"servePlaceIds": [123456789]
```

Only let the plugin connect from these place ids. `blockedPlaceIds` is the opposite list. `placeId` and `gameId` record where the project belongs.

### `serveAddress`

```luau
"serveAddress": "0.0.0.0"
```

Default bind address for `rojo serve`; `serveAllowedHosts` lists extra `Host`/`Origin` values to accept (7.7.0).

### `globIgnorePaths`

```luau
"globIgnorePaths": ["**/*.spec.luau", "!src/keep.spec.luau"]
```

Globs (relative to the project file) that Rojo skips entirely. A leading `!` re-includes a path, gitignore style (7.7.0).

### `emitLegacyScripts`

```luau
"emitLegacyScripts": true
```

Default `true`: `.server` files become `Script` (RunContext Legacy) and `.client` files become `LocalScript`. Set `false` to emit `Script` with RunContext `Server` or `Client` instead. Overwritten at every nested project file, so one that omits the key resets to `true`.

### `syncRules`

```luau
"syncRules": [{ "pattern": "*.md", "use": "text", "suffix": ".md" }]
```

Custom file rules checked before the defaults (7.5.0). `use` is one of serverScript, clientScript, moduleScript, pluginScript, json, jsonModel, project, toml, yaml, csv, text, rbxm, rbxmx or ignore (plus legacyServerScript, legacyClientScript, runContextServerScript and runContextClientScript, which force one script form); `exclude` is an optional glob and `suffix` the part stripped from the name (the extension when omitted). Cleared at every nested project file: repeat the rules in each one.

### `nested project scope`

```luau
$path → other.project.json
```

Reaching another project file builds a fresh context. `syncRules` are cleared and `emitLegacyScripts` is overwritten, so neither carries into (or out of) a Wally package's `default.project.json`; only `globIgnorePaths` accumulate, each relative to the project file that listed them.

### `$schema`

```luau
"$schema": "https://raw.githubusercontent.com/rojo-rbx/vscode-rojo/master/schemas/project.template.schema.json"
```

Accepted since 7.5.0 in project, model and meta files, so an editor's schema link is not an unknown key. Every other unrecognized top-level key is a hard parse error.

### `syncbackRules`

```luau
"syncbackRules": { "ignoreTrees": ["Workspace/Terrain"], "ignorePaths": ["**/*.rbxm"], … }
```

Controls `rojo syncback`: `ignoreTrees`, `ignorePaths`, `ignoreProperties` (class → property names), `syncCurrentCamera` (default false), `syncUnscriptable` (default true), `ignoreReferents` (default false), `createIgnoreDirPaths` (default true).

## Instance descriptions

Every object inside `tree` describes one instance. Keys starting with `$` configure it; every other key is a child named after the key.

### `$className`

```luau
"$className": "Part"
```

The instance's class. Optional when `$path` is given, or when the name can be inferred: services under `DataModel`, `StarterPlayerScripts`/`StarterCharacterScripts` under `StarterPlayer`, `Terrain` under `Workspace`.

### `$path`

```luau
"$path": "src/shared"  |  "$path": { "optional": "Packages" }
```

Pull in a file or directory (relative to the project file). Its contents become this instance and its children. The `optional` form is skipped silently when the path is missing (7.5.0).

### `$className + $path`

```luau
{ "$className": "ScreenGui", "$path": "src/gui" }
```

Allowed only when the path would be a `Folder`; the explicit class wins. Any other class from disk is an error: “If $className and $path are both set, $path must refer to a Folder.”

### `$properties`

```luau
"$properties": { "Anchored": true, "Size": [4, 1, 4] }
```

Property values, implicit (plain JSON that Rojo types from its API database) or explicit (`{ "Vector3": [4, 1, 4] }`). Applied over whatever the path provided. `Name` and `Parent` are refused: Rojo logs “Property 'Name' cannot be set manually, ignoring” and skips them.

### `$attributes`

```luau
"$attributes": { "Speed": { "Float64": 16 } }
```

Sets the instance's `Attributes` property (7.3.0), replacing any attributes the path provided. Attributes resolve on their own, without a class to consult: a plain bool, number or string works (a plain number becomes a `Float64`), and every other type — `Int32`, `Vector3`, `Color3`, enums — needs the explicit wrapper.

### `$ignoreUnknownInstances`

```luau
"$ignoreUnknownInstances": true
```

When true, instances in Studio that the project does not describe are left alone instead of deleted on sync. Defaults to `true` for nodes without `$path` and to `false` for nodes with one.

### `$id`

```luau
"$id": "spawn"
```

Names this instance so `Ref` properties elsewhere can point at it with `Rojo_Target_<Property>` attributes (7.5.0).

## Files to instances

Default sync rules, first match wins. The suffix is stripped to make the instance name; anything else is ignored.

### `*.server.luau`

```luau
Foo.server.luau → Script "Foo"
```

Also `.server.lua`. RunContext `Legacy` by default, `Server` with `emitLegacyScripts: false`.

### `*.client.luau`

```luau
Foo.client.luau → LocalScript "Foo"
```

Also `.client.lua`. Becomes `Script` with RunContext `Client` when `emitLegacyScripts` is false.

### `*.plugin.luau`

```luau
Foo.plugin.luau → Script "Foo" (RunContext Plugin)
```

For plugins; plain `.plugin.lua(u)` files since 7.5.0, and `init.plugin.lua(u)` on a directory only since 7.7.0 — on 7.5.x and 7.6.x that directory form silently did nothing.

### `*.luau`

```luau
Foo.luau → ModuleScript "Foo"
```

Also `.lua`. Any script that is not server, client or plugin is a ModuleScript.

### `directory`

```luau
Foo/ → Folder "Foo"
```

Unless it holds an init file or a `default.project.json` — which is checked first and beats every init file. An `init.meta.json` can change the class of a Folder.

### `init files`

```luau
Foo/init.luau → ModuleScript "Foo" with the other files as children
```

`init.luau`, `init.server.luau`, `init.client.luau`, `init.plugin.luau` (and `.lua` forms) or `init.csv` turn the directory into that instance. The first in that order wins and the rest produce no instance at all.

### `*.meta.json`

```luau
Foo.meta.json → extra data for "Foo"
```

`properties`, `attributes`, `ignoreUnknownInstances` and `id` for the sibling **file** of the same name; `init.meta.json` does the same for the directory it sits in and may also set `className` on a Folder. Never an instance by itself. It is read only beside a script, `.json`, `.toml`, `.yaml`, `.txt` or `.csv` file: beside a directory, a `.model.json`, an `.rbxm` or an `.rbxmx` it is ignored without a word, because those carry their own properties.

### `*.model.json`

```luau
Foo.model.json → any instance tree
```

`{ "className": …, "properties": {…}, "attributes": {…}, "children": [{ "name": …, … }] }`. The top-level name comes from the file; a `name` key there is ignored and logs “This field has been ignored since Rojo 6.0”. PascalCase `Name`, `ClassName`, `Properties` and `Children` are accepted as aliases, which is what most old tutorials use. A node may also declare an `id`, so a model's instances can be `Ref` targets.

### `*.project.json`

```luau
Foo.project.json → that project's tree
```

Nested projects compose: a `$path` to a project file (or a directory holding `default.project.json`) inserts its whole tree, under a fresh context (see “nested project scope”). Never point a project's `$path` at its own folder — `{ "tree": { "$path": "." } }` recurses until the stack overflows.

### `*.json / *.toml / *.yaml`

```luau
Foo.json → ModuleScript returning the data
```

Data files become ModuleScripts whose `Source` is `return { … }`. `.jsonc` since 7.7.0; `.yml` and `.yaml` since 7.6.0. TOML datetimes become strings.

### `*.txt / *.csv`

```luau
Foo.txt → StringValue, Foo.csv → LocalizationTable
```

Text becomes the `Value`; CSV must follow Roblox's localization table columns.

### `*.rbxm / *.rbxmx`

```luau
Foo.rbxm → the saved model
```

Binary or XML models exported from Studio are inserted as-is; their contents are not editable as text.

## Property values

Implicit values are plain JSON that Rojo types from the property's declared type; explicit values wrap the type name and skip validation.

### `scalars`

```luau
true  |  "text"  |  42  |  1.5
```

Bool, String, Int32/Int64, Float32/Float64 and Enum items by name (`"Voxel"`) are all implicit.

### `vectors and colors`

```luau
[512, 20, 512]  |  [0.639, 0.635, 0.647]
```

Vector2/Vector3 and Color3 (0–1 floats) are implicit arrays; `{ "Color3uint8": [163, 162, 165] }` uses 0–255.

### `UDim2`

```luau
{ "UDim2": [[1, 0], [0, 48]] }
```

Explicit only: `[[xScale, xOffset], [yScale, yOffset]]`. `UDim` is `{ "UDim": [scale, offset] }`.

### `CFrame`

```luau
[0, 5, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1]
```

Twelve numbers (position then rotation matrix) implicitly, or `{ "CFrame": { "position": [...], "orientation": [[...], [...], [...]] } }`.

### `Attributes`

```luau
"Attributes": { "Speed": { "Float64": 16 }, "Boss": true }
```

An attribute value is resolved on its own, so a plain bool, number or string is enough (a plain number becomes a `Float64`); anything else needs the explicit wrapper. `Tags` is a plain array of strings.

### `Font`

```luau
{ "family": "rbxasset://fonts/families/Inconsolata.json", "weight": "Regular", "style": "Normal" }
```

Implicit object form for `FontFace`.

### `explicit wrapper`

```luau
{ "Vector3": [0, 0, 0] }  |  { "Enum": 5 }  |  { "BrickColor": 194 }
```

Use when a property is too new for Rojo's database or has no implicit form (UDim, UDim2, BrickColor, NumberRange, NumberSequence, ColorSequence, Rect, Ray, Axes, Faces, BinaryString, PhysicalProperties).

### `Ref links`

```luau
"$id": "spawn"  …  "$attributes": { "Rojo_Target_Value": "spawn" }
```

Since 7.5.0 a `Ref` property (an ObjectValue's `Value`, a Model's `PrimaryPart`, a Weld's `Part0`) is set from files by naming the target with `$id` (or `id` in a meta file) and giving the referrer an attribute `Rojo_Target_<Property>` whose value is that id.

> Ids are project-wide strings; a `Rojo_Id` attribute on an instance marks it as a target the same way `$id` does, for instances that came from a place file or a model. The course's build shows the attribute unresolved; Studio shows the property.

## Around Rojo

The files the day-to-day loop reads that are not Rojo's own.

### `rokit.toml`

```luau
[tools]
rojo = "rojo-rbx/rojo@7.7.0"
```

Pins each tool per project; `rokit add` writes a line, `rokit install` restores every line on a fresh clone.

### `wally.toml`

```luau
[package] name/version/registry/realm  [dependencies]  [server-dependencies]  [dev-dependencies]
```

Dependencies are `scope/name@version`; each section installs into `Packages/`, `ServerPackages/` or `DevPackages/`, and `wally install` writes `wally.lock`.

> Commit the manifest and lockfile; ignore the folders and mount them behind optional paths.

### `sourcemap.json`

```luau
{ "name", "className", "filePaths", "children" }
```

The tree `rojo sourcemap` writes; luau-lsp reads it to resolve instance paths. Scripts only unless `--include-non-scripts`.

### `build.yml`

```luau
setup-rokit → wally install → rojo build -o … → upload-artifact
```

The CI order: tools, then packages, then the build, then keep the file; `rojo upload` with a secret key goes after, on merges.

> `CompeyDev/setup-rokit@v0.2.1` installs every tool in `rokit.toml`.

### `.gitignore`

```luau
/*.rbxl  /*.rbxlx  sourcemap.json  /Packages  /ServerPackages
```

Outputs and installs are ignored; the project file, `src/`, `rokit.toml`, `wally.toml` and `wally.lock` are committed.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Rojo](https://github.com/rojo-rbx/rojo) | CLI and Studio plugin | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [Rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager (installs pinned Rojo, Wally, luau-lsp…) | 1.2.0 | `curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh \| bash` |
| [Wally](https://github.com/UpliftGames/wally) | Package manager; installs into Packages/ and ServerPackages/ | 0.3.2 | `rokit add UpliftGames/wally` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Language server that reads rojo sourcemap output | 1.69.0 | `rokit add JohnnyMorganz/luau-lsp` |
| [setup-rokit](https://github.com/CompeyDev/setup-rokit) | GitHub Action that installs the tools pinned in rokit.toml | v0.2.1 | `uses: CompeyDev/setup-rokit@v0.2.1` |
| [vscode-rojo](https://github.com/rojo-rbx/vscode-rojo) | VS Code extension: start/stop serve, install the plugin | 2.1.2 | `VS Code Marketplace: evaera.vscode-rojo (publisher evaera)` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- `rojo init` files ending in `.lua`: since 7.4.0 the templates write `.luau`. Both extensions still sync.
- `aftman add rojo-rbx/rojo` or `foreman`: Rokit (`rokit add rojo-rbx/rojo`) is the maintained toolchain manager; it reads old `aftman.toml` and `foreman.toml` files.
- A required `name` at the top of every project: optional since 7.4.1; `default.project.json` inherits its folder's name.
- Comments are not allowed in project files: 7.6.1 accepts `//` and `/* */` comments plus trailing commas in project, meta and model JSON.
- `rojo serve` uses long polling: 7.7.0 switched the plugin protocol to websockets and MessagePack, so a matching plugin version is required.
- Rojo cannot go from Studio back to files: 7.7.0 added `rojo syncback --input place.rbxl`, driven by `syncbackRules`.
- `.client.lua` always means LocalScript: with `emitLegacyScripts: false` it becomes a `Script` with `RunContext = Client`.
- Tutorials that connect the plugin to port 34872 by editing Studio settings: the plugin has its own Connect panel with address and port fields.
- `init.server.lua` as the only way to write a plugin's entry: `.plugin.luau` (7.5.0) and `init.plugin.luau` on a directory (7.7.0) mark it `RunContext = Plugin`.
- Ref properties (`ObjectValue.Value`, `PrimaryPart`) can only be set in Studio: since 7.5.0 `$id` / `id` plus a `Rojo_Target_<Property>` attribute link them from files.
