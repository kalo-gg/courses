# Layout conventions

*A folder layout that keeps server, client, shared, GUI, assets and packages apart and maps cleanly.*

Rojo · The Rojo Workflow · 10 min

---

The `rojo init` layout scales further than it looks. Keep three code roots (`src/server`, `src/client`, `src/shared`), add `src/gui` for interface trees, `assets/` for models described as JSON or exported as `.rbxm`, and let Wally own `Packages/` and `ServerPackages/`. Each root maps to exactly one place in the game, so a file's path tells you where it runs.

```text
my-game/
├── default.project.json
├── rokit.toml  wally.toml  wally.lock
├── src/
│   ├── server/     → ServerScriptService.Server        (Scripts, server modules)
│   ├── client/     → StarterPlayer.StarterPlayerScripts.Client
│   ├── shared/     → ReplicatedStorage.Shared          (modules, config)
│   └── gui/        → StarterGui                        (ScreenGui directories)
├── assets/         → ReplicatedStorage.Assets          (.model.json, .rbxm)
├── Packages/       → ReplicatedStorage.Packages        (wally, optional)
└── ServerPackages/ → ServerScriptService.ServerPackages (wally, optional)
```

## Conventions that pay off

| Decision | Convention | Why |
| --- | --- | --- |
| Entry scripts | One `init.server.luau` per system folder | The folder is the Script; helpers are its children |
| Config | `.json` files in `src/shared` | Designers edit data without Luau; scripts `require` it |
| GUI | A directory per ScreenGui with `init.meta.json` | Layout in `.model.json`, behavior in `.client.luau` beside it |
| Big builds | Stay in Studio or ship as `.rbxm` | Text formats are for things people edit as text |
| Unknowns | `true` on Workspace, `false` on code roots | Builders keep their work; code is exactly what git says |

Each row is a habit with a reason. One `init.server.luau` per system folder means the folder *is* the Script and its helpers are children only it can see. Config as `.json` in `src/shared` means designers change numbers without touching Luau. A directory per ScreenGui, with `init.meta.json` for the class and `.client.luau` files beside the layout, keeps behavior next to what it drives. Big builds stay in Studio or ship as `.rbxm`, because text formats are for things people edit as text. And tests live beside the code they test — `Round.spec.luau` next to `Round/` — which is why a later module ignores them at build time instead of moving them.

**The arena's target tree**

```text
ReplicatedStorage.Shared.Util              ModuleScript      src/shared/Util.luau
ReplicatedStorage.Shared.Config            ModuleScript      src/shared/Config.json
ServerScriptService.Server.Round           Script            src/server/Round/init.server.luau
ServerScriptService.Server.Round.Scoring   ModuleScript      src/server/Round/Scoring.luau
StarterPlayer.StarterPlayerScripts.Client.Input   LocalScript   src/client/Input.client.luau
StarterGui.Hud                             ScreenGui         src/gui/Hud/init.meta.json
StarterGui.Hud.Controller                  LocalScript       src/gui/Hud/Controller.client.luau
ReplicatedStorage.Assets.Sword             Tool              assets/Sword.model.json
```

The exercise gives you the project file and the target tree above and asks for the file listing that produces it. Watch the two directory cases: `Round` needs a child, so it is a directory with an init file, and `Hud` needs a class, so its directory carries `init.meta.json`.

**Key points**

- Three code roots plus `gui`, `assets` and Wally's package folders cover a full game.
- A file's folder says where it runs; its name says what class it is.
- Keep unknowns on Workspace, delete them under code roots, and leave big builds in Studio or `.rbxm`.
