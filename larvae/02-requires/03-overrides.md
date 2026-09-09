# Overrides and mounts

*When one corner of the project has to build differently, and where larvae learns what lands where.*

Larvae · Rewriting Requires · 10 min

---

Real projects are rarely uniform. A legacy folder may need the instance form while everything else uses strings; a tools directory may run under Lune. `[requires.overrides]` retargets by glob, and the first match wins.

**Per-glob targets**

```toml
[requires]
target = "roblox-string"

[requires.overrides]
"legacy/**" = "roblox-instance"
"tools/**" = { target = "path" }
```

A bare string names a target; a table can also set the indexing style for that glob alone.

## Mounts

To rewrite a require, larvae has to know where a file lands in the DataModel. Usually it already does — a Rojo project file says exactly that, and larvae reads it. `[requires.mounts]` is the fallback for projects without one.

**Saying it by hand**

```toml
[requires.mounts]
"src/shared" = "@game/ReplicatedStorage/shared"
"src/server" = "@game/ServerScriptService"
```

Rarely needed. If you find yourself writing this beside a `default.project.json`, check that `[rojo] project` points at it.

**Where larvae reads mounts from**

```toml
[rojo]
project = "default.project.json"              # the source project larvae reads mounts from
build_project = ".larvae/build.project.json"  # the derived one it writes for the output
```

Both are found by name when unset. larvae reads yours, writes the derived one, and stops — serving is rojo's job.

Mounts are the bridge between the two ends of a require: the alias names a place in the DataModel, the mount says which directory lands there, and the rewriter needs both to turn `@game/ReplicatedStorage/shared/Util` into something relative to the file that asked. When the Rojo project already says it, saying it again in `[requires.mounts]` is a second source of truth, and the two will drift the first time someone moves a folder.

**Key points**

- `[requires.overrides]` retargets by glob: a bare string names a target, a table adds `indexing_style`; first match wins, so the specific glob goes first.
- Mounts map directories to DataModel paths; larvae reads them from the Rojo project `[rojo] project` names.
- `[requires.mounts]` is the fallback for a project with no Rojo file, not a second copy of one that exists.
- `[rojo] build_project` is where the derived project lands, inside `.larvae` — which is why `init` offers to gitignore it.
