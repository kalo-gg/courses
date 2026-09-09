# What is larvae?

*A Rust toolchain that owns the whole build, instead of four tools that each own a slice of it.*

Larvae · Larvae Foundations · 9 min

---

A typical Luau project runs stylua to format, selene to lint, darklua to transform, and luau-lsp in the editor. Four binaries, four config files, four ideas of which files count. **larvae** is one binary that does all of it from one `larvae.toml`.

| Command | Does |
| --- | --- |
| `larvae init` | Writes a starter config shaped by what the project already has |
| `larvae process` | Rewrites requires and applies transformations into an output directory |
| `larvae check` | Validates requires and syntax across the project, writing nothing |
| `larvae fmt` | Formats in place, with stylua's options under stylua's names |
| `larvae lint` | Reports suspicious code — 64 lints |
| `larvae analyze` | Type diagnostics in the terminal — by spawning `larvae-lsp` |
| `larvae bundle` | Collects the project into a single file |
| `larvae lsp` | Serves an editor over stdio: lint and format, no types |
| `larvae worm` | Manages extensions — worms, which get a module of their own |
| `larvae self` | Installs and updates larvae, and wires up your editor |

**Four configs become one**

```text
before                      after
├── stylua.toml             ├── larvae.toml
├── selene.toml             └── .luaurc      (aliases; larvae reads it)
├── .darklua.json
├── default.project.json    ├── default.project.json
├── build.project.json      └── .larvae/build.project.json   (derived, gitignored)
└── .luaurc
```

The second Rojo project file is the tell of a darklua pipeline: one points at the source, one at the build. larvae derives the second from the first.

> **"One binary" has a footnote**
>
> Two of those ten jobs are not in the `larvae` binary. The Luau **analyzer** lives in a second crate, `larvae-lsp`. `larvae lsp` on its own serves lints, formatting, code actions and the outline and **no types**; `larvae analyze` does nothing at all except find `larvae-lsp` and hand the arguments over. Everything else in this course is genuinely one binary reading one file — but plan on installing two.

> **You are already using it**
>
> Every code editor in these Kalo lessons is backed by `larvae-lsp`. The squiggles under a mistake, the hover types, the completions — that is the tool this course is about, running on Kalo's server as you read.

## The part most projects came for

Formatting and linting are the familiar half. The half that is harder to get elsewhere is the **require rewriter**: you write `require("@pkg/signal")` and larvae turns it into whatever the target runtime needs, checking on the way that the require is one the runtime could actually satisfy.

**One spelling in, one of three shapes out**

```luau
-- what you write, once, in every file that needs it
local Signal = require("@pkg/signal")

-- what larvae process writes depends on [requires] target:
--   roblox-string     Roblox's string require form
--   path              a filesystem path, for Lune
--   roblox-instance   an instance chain, indexed the way indexing_style says
```

The alias `pkg` is one line in `larvae.toml`. Move the folder and you change that line, not the project.

The ten commands sort into three groups. Four write files: `init`, `fmt`, `process` and `bundle`. Three only report: `check`, `lint` and `analyze` — plus `fmt --check`, which is `fmt` with its hands tied rather than a command of its own. Three serve or manage: `lsp` for your editor, `worm` for extensions, `self` for larvae itself. CI wants the reporting group and nothing from the first; your editor talks to the third without you typing anything.

**Key points**

- One binary reads one `larvae.toml` and does nine of the ten jobs; the Luau analyzer is the tenth and lives in `larvae-lsp`.
- Existing `stylua.toml` and `selene.toml` files are read first; `[fmt]` and `[lint]` layer over them.
- The require rewriter is the job the other tools do not do: one alias, three output shapes, checked on the way.
- `check`, `lint`, `analyze` and `fmt --check` write nothing — they are the CI group.
