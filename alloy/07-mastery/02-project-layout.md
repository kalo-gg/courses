# Project layout

*One `[mount]` table says where each folder lands, and `alloy build` writes the Rojo project, the sourcemap and the aliases from it.*

Alloy · The toolchain and mastery · 10 min

---

A Roblox project describes its tree three times. The Rojo project file says where each folder lands. The `.luaurc` gives the aliases. A sourcemap lets luau-lsp follow requires. Rename one folder and two of them go stale. Alloy has you write the tree once, in `alloy.toml`, and writes the rest itself.

## The [mount] table

**alloy.toml, as the init wizard writes a game**

```toml
[project]
name = "satchel"
runtime = "@game/ReplicatedStorage/Alloy"
sourcemap = true
source_of_truth = true
mount_aliases = true

[mount]
client = ["src/client", "@game/StarterPlayer/StarterPlayerScripts/Client"]
server = ["src/server", "@game/ServerScriptService/Server"]
shared = ["src/shared", "@game/ReplicatedStorage/Shared"]
```

Each entry is `alias = [path, place]`: a folder on disk and where it lands in the DataModel. `runtime` is where `alloy.luau` lands. The wizard writes the default out so you can see it and move it.

With a placed `[mount]` table, the table is the tree. `source_of_truth` has `alloy build` write `default.project.json`, plus `.alloy/build.project.json`, the same tree pointed at `build/`, which `rojo serve` reads. `sourcemap` writes `sourcemap.json`. `mount_aliases` makes each name an alias, so `import { startup } from '@shared/startup'` resolves, and the ship artifact rewrites it to the instance path. Roblox reads no `.luaurc`.

Two alias names belong to the compiler. `@game` names the Roblox services and `@alloy` names the runtime, so a `[mount]` entry with either name is an error. Aliases in `.config.luau` or `.luaurc` win over mount names, and `.config.luau` wins over `.luaurc`. Inside a folder, follow the style guide: keep the entry scripts short and start the game from `src/shared/startup.aly`.

**The style guide's layout**

```text
src/
  client/
    main.client.aly
    systems/
  server/
    main.server.aly
    Providers/
    systems/
  shared/
    startup.aly
    components.aly
    data/
    types/
```

An entry script imports the startup and calls it. Real constants go in a constants file, each with a `---` doc comment.

> **The rest of alloy.toml**
>
> `[build] in` and `out` default to `src` and `build`. `artifact` is `ship`, what Roblox runs, or `check`, which keeps the types for luau-lsp. `clean = true` also deletes every `.luau` under `out` that no source produces. The same keys can live in a `.config.aly` written in Alloy, and `alloy migrate` converts an `alloy.toml` into one.

**Key points**

- `[mount]` entries are `alias = [path, place]`. A bare path is an alias for a folder that another mount already carries.
- With a placed table and `source_of_truth`, `alloy build` writes `default.project.json` and `.alloy/build.project.json`. `sourcemap` writes `sourcemap.json`.
- `mount_aliases` makes `@shared/…` resolve, and the ship artifact rewrites the import to the instance path.
- `@game` and `@alloy` are reserved. Luau config aliases win over mount names, and `.config.luau` wins over `.luaurc`.
