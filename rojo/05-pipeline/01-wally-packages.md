# Packages with Wally

*Declare dependencies in wally.toml, install them into Packages folders, and mount those folders behind optional paths.*

Rojo · Packages, Syncback & CI · 10 min

---

Code you did not write arrives through **Wally**, the package manager most Rojo projects use: you list dependencies in a `wally.toml`, and `wally install` downloads them into `Packages/` (shared code) and `ServerPackages/` (server-only code). Because it is all plain files and directories (most packages carry their own `default.project.json`, which Rojo nests), Rojo syncs them like your own code. Point `ReplicatedStorage.Packages` at `Packages` with an optional path, run `wally install`, and the packages appear in Studio and in the sourcemap.

**wally.toml**

```toml
[package]
name = "you/arena"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
Promise = "evaera/promise@4.0.0"

[server-dependencies]
ProfileStore = "you/profilestore@1.0.0"
```

Install Wally itself with `rokit add UpliftGames/wally`. `realm` says where a package is allowed to end up: a `shared` package lands in `Packages/`, a `server` one in `ServerPackages/`, and a shared package may not depend on a server one. `wally install` writes both folders and a `wally.lock` that pins exact versions. `[server-dependencies]` lands in `ServerPackages/`; `[dev-dependencies]` in `DevPackages/`.

## Packages in the project file

`wally install` writes three things: the package folders, a `wally.lock` that pins every resolved version, and inside each folder an `_Index/` directory holding the real sources plus one thin link module per dependency — `Packages/Promise.luau` just requires `_Index/evaera_promise@4.0.0/promise`. Commit `wally.toml` and `wally.lock`; ignore the folders, because the lockfile recreates them exactly. Because a fresh clone has no `Packages/` until someone runs `wally install`, the project file mounts the folders behind **optional** paths, so the tree still builds — with the packages missing, and in silence — on a checkout that has not installed yet.

```json
"ReplicatedStorage": {
  "Packages": { "$path": { "optional": "Packages" } }
},
"ServerScriptService": {
  "ServerPackages": { "$path": { "optional": "ServerPackages" } }
}
```

> **Your project settings stop at the package boundary**
>
> Because each package brings its own `default.project.json`, Rojo builds it under a fresh context: your `syncRules` are cleared and your `emitLegacyScripts` is replaced by whatever that file says — which, since packages almost never set it, means `true`. So `emitLegacyScripts: false` gives you RunContext Scripts and leaves every package's `.client.luau` a LocalScript, and a `syncRules` entry for `.md` files stops working the moment you cross into `Packages/`. Only `globIgnorePaths` carry through.

`ServerPackages` can mount under `ServerScriptService` or `ServerStorage`; either is invisible to clients. What matters is that both mounts are optional, and that you remember what optional means: a missing folder is not an error, not a warning, just an absence.

**Key points**

- `wally.toml` lists `[dependencies]` (→ `Packages/`) and `[server-dependencies]` (→ `ServerPackages/`); `wally install` writes the folders and `wally.lock`.
- Commit the manifest and lockfile; ignore the folders.
- Mount package folders behind optional paths so a fresh clone still builds — silently without them.
- Each package's `default.project.json` resets `syncRules` and `emitLegacyScripts`; only `globIgnorePaths` carry through.
