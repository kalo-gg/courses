# Several places, one repository

*One project file per place, shared code mapped once, and settings that stop at each project file's edge.*

Rojo · Structuring a Real Game · 10 min

---

A game with a lobby and an arena is two places, and a repository serves both by holding two project files. `default.project.json` is what every command picks up when you name nothing; `lobby.project.json` is what you pass explicitly: `rojo serve lobby.project.json`, `rojo build lobby.project.json -o lobby.rbxl`. Both files point at the same `src/shared`, so shared code lives once, and each adds its own server folder. The one rule that bites: a project file you build or serve by name must have a `name`, because unlike `default.project.json` it has no folder name to fall back on.

**lobby.project.json**

```json
{
  "name": "lobby",
  "servePlaceIds": [2222222222],
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Shared": { "$path": "src/shared" },
      "Packages": { "$path": { "optional": "Packages" } }
    },
    "ServerScriptService": {
      "Lobby": { "$path": "src/lobby" }
    }
  }
}
```

## Shared code, separate settings

Sibling project files share nothing but the files they point at. `servePort`, `servePlaceIds`, `globIgnorePaths`, `emitLegacyScripts` and `syncRules` are read from the file you named and no other, so the lobby lists its own place id and repeats any ignore or sync rule it needs.

That separation is also the safety: `servePlaceIds` per file means the arena tree can never be synced into the lobby place by mistake. CI builds each place with its own command and keeps each artifact.

**Two places, two commands**

```bash
rojo build -o arena.rbxl                              # default.project.json
rojo build lobby.project.json -o lobby.rbxl
rojo serve lobby.project.json --port 34873            # alongside the arena on 34872
```

> **One project file per place**
>
> Keep one project file per place (`default.project.json` for the game, `lobby.project.json` for the lobby) so CI can build each with `rojo build lobby.project.json -o lobby.rbxl`. Shared code lives once in `src/shared` and every project file points at it. Give each of these a `name`: the project you hand to `build` or `serve` must have one unless it is a `default.project.json`, which borrows its folder's name. (A project reached through a `$path` is the exception — it falls back to its own file name — but a place you build by name is not.)

**Key points**

- One project file per place; `rojo serve <file>` and `rojo build <file> -o …` name it, and a named file needs `name`.
- Shared code is mapped from every project file; nothing else is shared between them.
- `servePlaceIds` per file keeps each tree out of the other place; two servers need two ports.
- CI builds each place with its own command.
