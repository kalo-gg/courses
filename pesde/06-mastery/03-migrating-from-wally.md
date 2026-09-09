# Migrating a Wally project

*wally.toml to pesde.toml, Packages to roblox_packages, and the three things that break on the way.*

pesde · Best Practices & Capstone · 11 min

---

A Wally project is already most of a pesde project: a manifest, a lockfile, a packages directory and a project file that mounts it. The migration is a rename of each of those, plus one script Wally never needed. Nothing has to move off Wally on day one — every Wally package stays a Wally dependency until its author publishes to pesde.

| Wally | pesde |
| --- | --- |
| `wally.toml` | `pesde.toml` |
| `signal = "sleitnick/signal@2.0.0"` | `signal = { wally = "sleitnick/signal", version = "^2.0.0" }` |
| `[server-dependencies]` | `roblox_server` target — for private registries only |
| `Packages/`, `ServerPackages/` | `roblox_packages/`, `roblox_server_packages/` |
| `wally install` | `pesde install` |
| `wally.lock` | `pesde.lock` |
| `default.project.json` inside a package | `build_files` in `[target]` |
| `realm = "server"` | `environment = "roblox_server"` |

The left column is Wally's own spelling, checked against Wally's manifest format: a `[server-dependencies]` table beside `[dependencies]`, and `realm = "server"` (the other value being `"shared"`) on the package itself.

**A migrated game manifest**

```toml
name = "you/arena"
version = "0.1.0"
private = true

[target]
environment = "roblox"

[wally_indices]
default = "https://github.com/UpliftGames/wally-index"

[dependencies]
signal = { wally = "sleitnick/signal", version = "^2.0.0" }
promise = { wally = "evaera/promise", version = "^4.0.0" }

[dev_dependencies]
scripts = { name = "pesde/scripts_rojo", version = "^0.2.0", target = "lune" }
rojo = { name = "pesde/rojo", version = "^7.5.1", target = "lune" }

[scripts]
roblox_sync_config_generator = ".pesde/scripts/roblox_sync_config_generator.luau"
sourcemap_generator = ".pesde/scripts/sourcemap_generator.luau"
```

A game is `private`, exports nothing, and keeps its Wally packages as Wally specifiers. `lib` and `build_files` are publish-time requirements, not install-time ones, so this manifest installs without a word of complaint — the refusals it would trip are never reached, because `private = true` stops publish first.

## The three things that break

First, names. Wally allows hyphens; pesde treats one as a Wally marker, so a package you *publish* must be renamed with underscores — dependencies you consume from Wally keep theirs. Second, requires. `Packages.Signal` becomes `roblox_packages.signal`: the linker is named after the alias, and the default alias is the package name without its scope.

Third, the sourcemap. Wally packages have no manifest pesde can read, so without `[scripts] sourcemap_generator` every one of them resolves to nothing — the migration's most common 'it installed but does not work'. It is also the one of the three that no error tells you about: the message is a warning about types, and the damage is the entry point.

**What the requires become**

```luau
-- before: Wally's Packages folder, capitalised package names
local Signal = require(ReplicatedStorage.Packages.Signal)
local Promise = require(ReplicatedStorage.Packages.Promise)

-- after: pesde's roblox_packages, keyed by your aliases
local Signal = require(ReplicatedStorage.roblox_packages.signal)
local Promise = require(ReplicatedStorage.roblox_packages.promise)
```

The folder is renamed and its children are the aliases from `[dependencies]` — the default alias is the package name without its scope. Where that folder sits in the datamodel is your project file's business, and `[place]` is how you tell pesde.

**Key points**

- Rename the files and directories; keep Wally packages as `wally = …` specifiers until they exist on pesde.
- Hyphens only matter for names you publish.
- Requires go through `roblox_packages.<alias>`, and the default alias is the unscoped name.
- Without a `sourcemap_generator`, a migrated Wally dependency installs and resolves to nothing.
