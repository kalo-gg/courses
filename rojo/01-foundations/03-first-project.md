# Your first project

*rojo init, build a place, serve it, connect the plugin, and watch a file change land in Studio.*

Rojo · Rojo Foundations · 11 min

---

`rojo init my-game` creates a folder with a working place project. It is small enough to read in full, and every later project is a variation on it.

**What rojo init creates**

```text
my-game/
├── default.project.json      the project file
├── README.md
├── .gitignore                ignores the built place and sourcemap.json
└── src/
    ├── client/init.client.luau   → LocalScript "Client"
    ├── server/init.server.luau   → Script "Server"
    └── shared/Hello.luau         → ModuleScript "Hello"
```

**default.project.json (trimmed)**

```json
{
  "name": "my-game",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Shared": { "$path": "src/shared" }
    },
    "ServerScriptService": {
      "Server": { "$path": "src/server" }
    },
    "StarterPlayer": {
      "StarterPlayerScripts": {
        "Client": { "$path": "src/client" }
      }
    },
    "Workspace": {
      "$properties": { "FilteringEnabled": true },
      "Baseplate": {
        "$className": "Part",
        "$properties": { "Anchored": true, "Locked": true, "Position": [0, -10, 0], "Size": [512, 20, 512] }
      }
    }
  }
}
```

Three folders map into three services; Workspace also gets a baseplate described entirely in the project file. The real template adds Lighting and SoundService settings.

## Build, then serve

```bash
cd my-game
rojo build -o my-game.rbxlx   # a place file you can open in Studio (.rbxl is the binary form)
rojo serve                    # Rojo server listening: Address: localhost  Port: 34872
```

Open the built place in Studio, click the Rojo plugin button and press **Connect** (the defaults, `localhost` and `34872`, match the server). Before it changes anything, the plugin shows a confirmation listing every instance it will add, update or **remove** to make the place match your files; read it, then accept. From now on every save under `src/` shows up in the Explorer. Try it: change the string in `src/shared/Hello.luau` and watch `ReplicatedStorage.Shared.Hello` update. The build step is only needed the first time, or whenever you want a fresh file for a release. Saving and publishing the place is still done from Studio (or with `rojo upload`, in Publishing places and plugins).

> **Sync is one way**
>
> While connected, files win. An edit made in Studio's script editor is overwritten the next time the matching file is saved, so edit scripts in your editor and use Studio for building, testing and playing. (The plugin has an experimental two-way sync setting; leave it off until you know why you want it.) Anything the project does not describe, such as a map you build by hand, is left alone.

> **Already have a game?**
>
> Connecting to an existing place is the same flow, but read the confirmation carefully: instances under a synced folder that have no file are removed. What Rojo owns explains which instances Rojo owns, and Syncback pulls an existing place into files so you can adopt a Studio-built game. With Team Create, only one person should be connected at a time; synced changes replicate to everyone like any other edit.

Before the exercise, three rules about `tree` that the next lesson explains in full: every key that does not start with `$` is a child instance named after the key; `"$path": "folder"` pulls that folder's files in as the instance's contents; and `//` comments are allowed in Rojo's JSON files, which the starters use for TODOs.

**Key points**

- `rojo init` creates `default.project.json` plus `src/client`, `src/server` and `src/shared`.
- `rojo build -o file.rbxlx` makes a place file; `rojo serve` plus the plugin's Connect keeps the open place updated.
- In `tree`, keys are instance names, `$className` sets a class, and `$path` pulls in a folder.
