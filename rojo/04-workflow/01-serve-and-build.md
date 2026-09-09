# Serve and build

*rojo serve options and project settings, rojo build output formats and --watch.*

Rojo · The Rojo Workflow · 9 min

---

`rojo serve [project]` starts the sync server. The project argument is a directory (Rojo looks for `default.project.json` inside) or a specific `.project.json`, which is how one repository serves several places: `rojo serve lobby.project.json`. The port comes from `--port`, then the project's `servePort`, then 34872; the address from `--address`, then `serveAddress`, then `127.0.0.1`.

```bash
rojo serve                          # default.project.json, 127.0.0.1:34872
rojo serve lobby.project.json --port 34873
rojo serve --address 0.0.0.0 --allowed-hosts mypc.lan   # for Studio on another machine
```

Binding beyond localhost prints a warning: the API is unauthenticated. `--allowed-hosts` (7.7.0) adds extra `Host`/`Origin` values the server accepts beyond localhost and the bind address, replacing the project's `serveAllowedHosts`.

Two project settings protect people from syncing into the wrong place. `servePlaceIds` lists the place ids the plugin may connect from; `blockedPlaceIds` is the opposite. Put your test place in `servePlaceIds` and nobody can accidentally sync the dev tree into production.

## Building files

`rojo build -o out.rbxl` writes the whole tree to disk without Studio. The extension chooses the format: `.rbxl` and `.rbxlx` are places (binary and XML) and Rojo writes the DataModel's children; `.rbxm` and `.rbxmx` are models and include the root instance itself, which is what a library project with a `Folder` or `ModuleScript` root wants. `--watch` rebuilds on every change, handy when Studio is not around.

**Three builds**

```bash
rojo build -o game.rbxl                       # binary place, the release artifact
rojo build --watch -o game.rbxlx              # XML place, rebuilt on every change
rojo build lib.project.json -o lib.rbxm       # a model: the root instance is included
```

| Command | Produces |
| --- | --- |
| `rojo build -o game.rbxl` | Binary place: the usual release artifact |
| `rojo build -o game.rbxlx` | XML place: diffable, larger |
| `rojo build -o lib.rbxm` | Binary model of the root instance: for libraries and plugins |
| `rojo build --plugin MyPlugin.rbxm` | Installs a plugin project into Studio's plugins folder |

**Key points**

- `rojo serve [project]` picks port and address from flags, then the project file, then defaults (34872, 127.0.0.1).
- `servePlaceIds` and `blockedPlaceIds` stop syncing into the wrong place.
- `rojo build -o file` chooses place or model output from the extension; `--watch` rebuilds on change.
