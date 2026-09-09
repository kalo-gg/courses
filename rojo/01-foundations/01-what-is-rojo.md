# What is Rojo?

*A CLI and a Studio plugin that keep a folder of files and a Roblox place in sync.*

Rojo · Rojo Foundations · 8 min

---

Open Studio, edit a script, save the place. That loop works until the day two people edit the same script, a bad change needs undoing from last week, or you want to run a formatter, a linter or tests. Studio keeps everything inside one place file that only Studio can read, so none of the tools programmers rely on elsewhere can reach your code.

**Rojo** turns that around. Your scripts live as plain files in a folder: `Shop.server.luau`, `Inventory/init.luau`, `Config.json`. A small **project file** says where those folders land in the game. Rojo then does the mapping for you: live, files into Studio, while you work, and back the other way with an offline command when you need it.

**Three files, three instances**

```text
src/server/Shop.server.luau      → ServerScriptService.Server.Shop      (Script)
src/shared/Inventory/init.luau   → ReplicatedStorage.Shared.Inventory   (ModuleScript)
src/shared/Config.json           → ReplicatedStorage.Shared.Config      (ModuleScript)
```

Left of the arrow is what you edit; right of it is what Studio shows. The project file decides the service, the file name decides the class.

| Command | Direction | What you get |
| --- | --- | --- |
| `rojo serve` | files → Studio, live | A local server; the Studio plugin connects and applies every save within a second |
| `rojo build -o game.rbxl` | files → place file | A complete `.rbxl`/`.rbxm` built from scratch, ideal for releases and automated builds (CI) |
| `rojo syncback --input game.rbxl` | Studio → files | New in 7.7.0: pull instances from a saved place back into your folders |

## Two halves

Rojo is two programs. The **command-line tool** (`rojo`) reads your files, builds the instance tree and, when serving, listens on a local port. The **Studio plugin** connects to that port, receives the tree and creates or updates instances in the open place. By default the plugin never touches your files, and the CLI never touches Studio; they only talk over the local connection. 7.7.0 changed how the two talk to each other, and the plugin refuses any CLI whose protocol version is not an exact match — newer or older — so keep them in step.

**The two halves talking**

```bash
$ rojo serve
Rojo server listening:
  Address: localhost
  Port:    34872

Visit http://localhost:34872/ in your browser for more information.
# Studio → Plugins tab → Rojo → Connect (localhost, 34872)
```

The CLI prints where it listens; the plugin's Connect panel asks for the same address and port.

> **What stays in Studio**
>
> Rojo is great at scripts, folders, values and small models described as text. Terrain, large builds and anything you sculpt by hand are usually left in Studio; the project file simply does not describe them, and Rojo leaves what it does not describe alone (`$ignoreUnknownInstances`, in What Rojo owns).

**Key points**

- Rojo keeps game code as files on disk and maps them onto Roblox instances.
- `rojo serve` syncs live into Studio; `rojo build` writes a place or model file; `rojo syncback` (7.7.0) goes the other way.
- The CLI reads files and serves the tree; the Studio plugin applies it. Their versions must match.
- Studio keeps whatever the project does not describe: terrain and hand-built maps usually stay there.
