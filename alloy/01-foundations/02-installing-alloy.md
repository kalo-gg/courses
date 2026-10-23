# Installing and building

*Install the binaries, let `alloy init` write the project, and keep `alloy build -W` compiling.*

Alloy · Foundations · 9 min

---

Alloy runs on your machine, not in the game. Two binaries do the work: `alloy`, the compiler and CLI, and `alloy-lsp`, the language server your editor talks to. A project then needs one file that says where its sources are and where the Luau goes.

## Install, then init

**Installing**

```bash
cargo install alloy-luau alloy-luau-lsp

# or, from a release zip:
alloy self install    # copies alloy and alloy-lsp into ~/.alloy/bin
alloy self update     # later: install a newer release
```

The crates are `alloy-luau` and `alloy-luau-lsp`; the binaries they install are `alloy` and `alloy-lsp`. Each release carries one zip per binary per platform.

`alloy init` in a terminal opens a wizard. Its first question is `Use the recommended setup?`, and yes writes a game: `src/client`, `src/server` and `src/shared`, strict mode, the `temper` style preset, Ember, Lest and an `AGENTS.md`. `alloy init --yes` writes the same with no prompt. A script, a pipe or `--non-interactive` gets only the plain files.

Either way it writes `alloy.toml` and `.alloy/alloy.schema.json`, plus a `.config.luau` in strict mode when the folder has no Luau config yet.

**Terminal**

```bash
alloy init         # the wizard; answer yes
alloy build -W     # compile src into build, again on every change
```

## alloy.toml

**alloy.toml (from the recommended setup, shortened)**

```toml
#:schema .alloy/alloy.schema.json
[build]
in = "src"
out = "build"
exclude = []
clean = false
artifact = "ship"

[emit]
wait_timeout = 5

[std]
globals = "none"

# … [fmt], [lint], [flux] and [test] …

[project]
name = "arena"
runtime = "@game/ReplicatedStorage/Alloy"
sourcemap = true
source_of_truth = true
mount_aliases = true

[mount]
client = ["src/client", "@game/StarterPlayer/StarterPlayerScripts/Client"]
server = ["src/server", "@game/ServerScriptService/Server"]
shared = ["src/shared", "@game/ReplicatedStorage/Shared"]
```

`[mount]` says where each folder lands in the game, and `source_of_truth` writes the Rojo project from it. Every key has a default, and an unknown key is an error.

`alloy build` compiles every `.aly` and `.alx` under `[build] in` into `out`, mirroring the tree, and writes the runtime there as `alloy.luau`. A file with an error is not written, so its last output stays. `-W` builds again on every change. `clean = true` also deletes any `.luau` under `out` that no source makes. Rojo syncs the output, never `src`.

In VS Code, the Alloy extension starts `alloy-lsp` from your PATH.

> **Configuration in Alloy**
>
> A project can write the same keys in Alloy instead: a `.config.aly` that exports them, as in `export const build = { out = 'dist' }`. `alloy.toml` is the default, and `alloy migrate` rewrites one as a `.config.aly`. Module 7 covers it.

**Key points**

- `cargo install alloy-luau alloy-luau-lsp`, or a release zip and `alloy self install`, which copies both binaries into `~/.alloy/bin`.
- `alloy init` opens the wizard in a terminal; `--yes` writes the recommended setup with no prompt, and a pipe gets only the plain files.
- `[build] in` and `out` say what compiles and where; the runtime lands at the output root as `alloy.luau`.
- `alloy build -W` rebuilds on every change, and Rojo syncs the output.
