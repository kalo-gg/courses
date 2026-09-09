# Larvae reference

*Larvae 0.8.1, released September 4, 2026. Checked on September 7, 2026.*

---

## Commands

Ten jobs, one larvae.toml. Nine of them live in the larvae binary; `analyze` spawns a second one.

### `larvae init`

```luau
larvae init
```

Writes a starter `larvae.toml` shaped by what the project already has, and offers to update `.gitignore`. With a `stylua.toml` or `selene.toml` present it writes a comment pointing at that file instead of a `[fmt]` or `[lint]` table; without one it writes `recommended = true` plus the four options projects change most.

> It hands linting over in `.luaurc` only when there is a `.luaurc` that does not already set `lint`. No file, or an opinion already written, and it leaves the project alone and says so.

### `larvae process`

```luau
larvae process
larvae process --watch
larvae process --profile release
```

Reads the input directory, rewrites every require into the configured target, applies the enabled rules, and writes the result to the output directory — plus `.larvae/build.project.json`, the derived Rojo project you serve.

> The job most projects came for. Needs at most `[process]` and `[requires]`.
>
> `--watch` rebuilds on every save and mirrors deletions; a file that does not lex keeps its last good output, so a half-typed save does not spread failures into a live Studio session.

### `larvae check`

```luau
larvae check
```

Validates requires and syntax across the whole project without writing anything. The CI gate: unresolvable requires, realm violations, alias cycles and syntax errors. It also counts the dynamic requires it deliberately did not touch.

### `larvae fmt`

```luau
larvae fmt
larvae fmt --check
larvae fmt --stdin --stdin-filepath a.luaux
```

Formats Luau in place. `--check` writes nothing and exits non-zero if a file would change, which is the CI invocation; `--stdin` formats one file through the pipes, with `--stdin-filepath` so a worm can claim it.

> A `stylua.toml` is read first and `[fmt]` layers over it — but the defaults are not stylua's. Three options carry larvae's own opinion and are on by default. `[fmt] recommended = false` turns exactly those three off.

### `larvae lint`

```luau
larvae lint
larvae lint --explain <name>
larvae lint --stdin
```

Reports suspicious code without changing it. `--explain` describes any one of the 64 lints, worm lints included.

### `larvae analyze`

```luau
larvae analyze [paths…] --definitions <file>
```

Type diagnostics in the terminal, from the same analyzer the editor uses. The command has two halves: it finds the `larvae-lsp` binary and hands the arguments over, so it does nothing at all without that binary installed.

### `larvae bundle`

```luau
larvae bundle --entry <file> --out <file>
```

Collects the project into a single file. Modules load on demand, so bundling cannot move a side effect.

### `larvae lsp`

```luau
larvae lsp
```

Serves an editor over stdio — lint, format, code actions and the outline, and no types. Hover, completion and type diagnostics come from the separate `larvae-lsp` binary, which plugs Luau's analysis frontend into the same server.

### `larvae worm`

```luau
larvae worm <run|info|types|add|install|claims|remove>
```

`add` writes a worm into `larvae.toml` at its newest release (`--cargo` takes it from crates.io); `install` (alias `i`, with `--force`) downloads every worm the config names; `remove` (alias `rm`) deletes both. `run <dir> <file>` passes one file through a worm directory with `--fmt` or `--lint`, `info <dir>` reports what a worm declares, `types` writes `worm.d.luau`, and `claims` prints the extensions this project's worms claim as JSON.

> There is no `larvae worm develop`. `run`, `info` and `types` are the development loop, and they need no project and no install.
>
> `larvae worm add luaux` takes a short name larvae knows and expands it to `larvae-luau/luaux-worm` with `version = "^"`, writing the table for you.

### `larvae self`

```luau
larvae self <install|update|uninstall|code|sync-luaurc>
```

`install` copies the binary to `~/.larvae/bin` and fetches `larvae-lsp` and its analyzer library when they are not beside it — which is what a rokit or aftman user needs, because a tool manager installs only the binary its manifest names. `code` sets up schema completion and hover on `larvae.toml`. `sync-luaurc [--check]` writes the project's aliases into `.luaurc` as paths an editor can follow.

> Two of the five are not installation management at all. `larvae self code` is what makes `larvae.toml` complete as you type it.

## Requires: the rewriter

Aliases in, one of three shapes out, with the realm checked on the way.

### `[requires] target`

```luau
target = "roblox-string"   # or "path", "roblox-instance"
```

What a require becomes. `roblox-string` writes Roblox's string form, `path` writes filesystem paths for Lune, and `roblox-instance` writes an instance chain.

### `[requires] indexing_style`

```luau
indexing_style = "find_first_child"   # the default
# or "wait_for_child", "property"
```

How an emitted instance chain indexes each child, for `roblox-instance` only. The default is `find_first_child`, which does **not** wait — which is why `[check] early_require` warns by default.

> `larvae.example.toml` lists `wait_for_child` first, so it reads like the default. It is not.
>
> See `[check]` in *The gate & CI*: `early_require` is the finding this default produces, and the two are taught in different lessons of the same module.

### `[requires] cross_realm`

```luau
cross_realm = "deny"   # or "allow"

-- larvae: allow(cross_realm_require)
```

Whether a require reaching across the halves of the game stops the run. Denied by default, because a client requiring a server module compiles fine and fails at runtime. The per-line allow comment is the escape the project recommends; `allow` here turns the check off everywhere.

> The realm check is the thing larvae does that a plain formatter cannot.

### `[requires] client_relative_requires`

```luau
client_relative_requires = false   # the default
```

Writes a require inside the StarterPlayer tree as a relative path, so the script reaches its siblings *after* Roblox clones the container into the player. Off by default, because it lowers the floor of a relative walk from the mount to the container. A shared file requiring client code stays an error either way.

### `[aliases]`

```luau
[aliases]
pkg = "@game/ReplicatedStorage/Packages"
shared = "./src/shared"
```

Merged over `.luaurc` key by key, so a project that already has one needs none of this. Values are DataModel or filesystem paths; `@self` and `@game` are reserved.

### `[requires.overrides]`

```luau
[requires.overrides]
"legacy/**" = "roblox-instance"
"tools/**" = { target = "path" }
```

A different target for the files a glob matches. First match wins — the escape hatch for a corner of the project that has to build differently.

### `[requires.mounts]`

```luau
[requires.mounts]
"src/shared" = "@game/ReplicatedStorage/shared"
```

Filesystem to DataModel mapping. Rarely needed: a Rojo project file already says this and larvae reads it.

### `[rojo]`

```luau
[rojo]
project = "default.project.json"
build_project = ".larvae/build.project.json"
```

The source project file larvae reads mounts from, and the derived one it writes for the build output. larvae never runs rojo: it writes the second file and stops, so `rojo serve .larvae/build.project.json` is the last line of the loop.

> Both are found by name when unset. This is the one project file instead of two that the README argues for against darklua.

## Formatting

Every stylua option under its own name — and three opinions that are not stylua's.

### `[fmt] recommended`

```luau
[fmt]
recommended = false   # absent and true both mean larvae's defaults
```

`false` starts from a base that changes as little as possible: `magic_trailing_comma`, `space_inside_braces` and `trailing_comma` go back to off, and every other default was stylua's already. This is the switch that makes stylua parity literally true, and it is the switch to reach for on the first day of a migration.

> It is in neither `larvae.example.toml` nor the README, but `larvae init` writes `recommended = true` into the config it generates, so a new project sees the key even though the reference file does not carry it.

### `Past stylua`

```luau
magic_trailing_comma = true
space_inside_braces = true
trailing_comma = true
block_newline_gaps = "never"
```

The three `true`s are larvae's own opinion, on by default because the Luau community already writes that way — and they will change your files on the first run. `magic_trailing_comma` makes the author's trailing comma mean "keep this table expanded", which also means a hand-aligned table with a trailing comma is *exploded* to one entry per line rather than collapsed.

### `[fmt] unused_imports`

```luau
unused_imports = "ignore"   # or "underscore", "remove"
```

`ignore` by default, because `require` runs a module the first time anything asks for it: deleting an apparently unused import can stop work the program relies on. A name only a type annotation reads still counts as used.

### `[fmt.sort_requires]`

```luau
[fmt.sort_requires]
enabled = false
grouping = "flat"   # or "by-kind"
```

Sorts the require block. `flat` is one sorted run, which is what stylua does; `by-kind` writes aliases, then absolute paths, then relative paths, with a blank line between the groups. Off by default — it moves lines, so it is opt-in.

### `It would rather decline than delete a comment`

```luau
formatting would drop the comment "…", so the file was left alone
```

The formatter checks every comment against its own output at runtime. A file whose formatting would lose one is left byte for byte as it was and the run reports it, rather than silently dropping a note. A file that mysteriously did not format is usually this.

### `Excludes`

```luau
exclude = ["Packages"]      # root: every command
include = ["Packages/keep.luau"]

[fmt]
exclude = ["**/*.gen.luau"]
```

Four levels of it: the root pair every command inherits, and `[process]`, `[fmt]` and `[lint]` each with their own. A match on any directory on the way to a file counts, so a bare directory name is enough. The root `include` cancels the root `exclude` and only that; the `include` of one area is stronger and cancels for that area too.

> Excludes never apply to a file named on the command line. A formatter that silently does nothing to a file you pointed at is worse than one that formats an excluded file.

## Linting

Sixty-four lints, and a level for each. Only deny fails a run.

### `Levels`

```luau
allow | info | warn | deny
```

`info` reports and leaves the exit code alone, drawn as a hint rather than a squiggle. `warn` is the same for the exit code but louder. Only `deny` fails the run. There is no `error` level: selene's `error` is spelled `deny` here.

### `The defaults`

```luau
6 deny · 11 allow · 47 warn
```

A lint denies only when the code cannot be what the author meant and no reading makes the finding wrong — `undefined_variable`, `duplicate_keys`, `duplicate_local`, `format_string`, `zero_step_loop`, `length_as_condition`. Everything softer warns, and a matter of taste is `allow` until asked for.

> Counted from `lint/lints/mod.rs`, whose `ALL` holds 64 entries. `larvae.example.toml` says sixty-three and the README says thirty-four; both are stale, and the set grows every release.

### `[lint] recommended`

```luau
[lint]
recommended = false

[lint.rules]
shadowing = "warn"
```

`false` starts every lint at `allow`, so a project gets the lints it names and no others. A level the project wrote wins in either state, which is the whole point: one lint on and the other sixty-three off.

### `[lint.groups]`

```luau
[lint.groups]
performance = "info"
complexity = "allow"
```

A level for a whole kind: correctness (29), suspicious (16), style (12), performance (3), roblox (3), complexity (1). A named lint always beats a group, and a group never wakes a lint that is `allow` on purpose.

### `[lint.options.<lint>]`

```luau
[lint.options.unused_variable]
ignore_pattern = "^_"

[lint.options.restricted_globals]
loadstring = "ship the code instead"
```

Per-lint settings — selene calls the same table `config`, and that spelling is accepted. `restricted_globals` and `restricted_module_paths` take a name and the reason a reader is shown; `deprecated.additional` maps an old name to its replacement; `unused_variable` takes `parameters`, `loop_variables` and `ignore_pattern`.

### `Luau parity`

```luau
28 of the 64 are Luau's own
```

larvae covers all 28 lints of the Luau compiler and keeps the level Luau gives each one. Where Luau consults the type checker and larvae cannot, larvae reports the narrower case that needs no types — so a few type-dependent findings are lost by design, and no false ones are gained. `larvae init` turns Luau's own off in `.luaurc` so nothing reports twice.

> Sixteen of them live in `lints/luau.rs`; the other twelve are covered by larvae lints under selene's spellings, such as `undefined_variable` and `shadowing`.

### `selene compatibility`

```luau
[lint]
std = "roblox"
globals = ["shared"]
```

selene's names work here, and an existing `selene.toml` is read first with `[lint]` layered over it. `-- selene: allow(...)` comments are honored too.

### `Flag comments`

```luau
local x = 1 -- larvae: allow(unused_variable, shadowing)
local y = 2 -- selene: allow(shadowing)        -- read too

-- larvae: fmt off        -- to the end of the file, with no "on"
-- larvae: fmt on         -- or a region, between the two
-- larvae: lint off(3)    -- or a count: the next three lines
```

How one file tells larvae about its own exception without changing the config for everyone. `allow(...)` takes one or more comma-separated lint names on the line it applies to; `fmt off` and `lint off` come in three shapes — to the end of the file, between an `off` and an `on`, or `off(n)` for a count of lines.

> selene's `-- selene: allow(...)` spelling is honored, so a migration rewrites no comments. Recognition is deliberately narrow because `larvae process` strips flag comments from the output (`strip_flags`) and must never delete a note. A worm's lint is dotted here, as everywhere outside `[lint.rules.<worm>]`: `-- larvae: allow(luaux.useless_fragment)`. The realm check is plumbed through the same vocabulary — `-- larvae: allow(cross_realm_require)`.

## Processing and rules

### `[process] input / output / include / exclude`

```luau
[process]
input = "src"                  # a directory, or a list of them
output = "dist"                # a mirror of input with the transforms applied
include = ["**/*.luau"]        # which found files are transformed; the rest are copied
exclude = ["**/*.spec.luau"]   # skipped fully: not transformed, not copied
```

What the build reads and writes. Two lists ask two questions: `include` decides which found files are transformed, `exclude` decides which are not found at all. A file matching neither is copied, because a build is a mirror of the tree rather than a filter.

> `input`, `output` and `target` also exist as root short forms, and writing both a short form and its table key is an error. `quotes` sets the quote style of the require strings the rewriter writes; `strip_flags` (default true) drops `-- larvae: allow(...)` comments from the output.

### `[bundle]`

```luau
[bundle]
entry = "src/plugin/init.luau"
output = "build/LanternPlugin.luau"    # default "bundle.luau"
tree_shake = true
module_ids = "paths"                   # or "opaque"
```

Single-file output for `larvae bundle`. The bundler walks out from `entry` following requires — that walk *is* the tree-shaking — and registers each module under a readable path or an opaque number.

> A dynamic require is the one thing the walk cannot follow, which is why the count `larvae check` prints is worth watching. Bundling changes how code is delivered, not when it runs: modules in the bundle still load on demand, so it cannot move a side effect. `larvae bundle --entry <file> --out <file>` overrides the table for one run.

### `extends`

```luau
extends = "./shared/larvae.base.toml"
```

A base config by path, relative to this file. The base loads first and this file wins key by key — how a monorepo keeps its shared twenty lines in one place.

### `[process] generator`

```luau
generator = "retain-lines"   # or "dense", "readable"
```

`retain-lines` keeps line numbers so a stack trace points at the source; `dense` minifies, tuned by `[minify]`; `readable` formats the output.

### `[minify]`

```luau
[minify]
column_span = 120
rename_variables = false
obfuscate = false
```

Tuning for `generator = "dense"`, inert under any other generator — except `obfuscate`, which turns the dense generator on by itself. `column_span` breaks the line near a column so a crash report still says something; `rename_variables` is the `[rules]` rule of the same name, reachable without editing `[rules]`.

> `obfuscate` drops every type, gives every local a `_0x` name, writes every string as its `\xNN` bytes, and lands the file on one line. Roblox removed `loadstring`, so what ships must still be Luau the compiler reads; names and strings are all that can be hidden.

### `[process] cache`

```luau
cache = true
cache_dir = ".larvae"
```

Incremental builds keyed by content hash. `.larvae` is also where the derived Rojo project lands, which is why `larvae init` offers to add it to `.gitignore`.

### `larvae's own rules`

```luau
const_requires · use_get_service · dedupe_requires
freeze_module · add_luau_directive · inject_module_path
remove_comments · remove_calls · append_text_comment
```

Transformations larvae adds on top of the darklua parity set — every one off unless named in `[rules]`.

### `darklua parity`

```luau
compute_expression · convert_index_to_field
remove_unused_variable · filter_after_early_return · …
```

The darklua rule set under darklua's own names, so an existing pipeline moves across without relearning it. Three moved and say so by name: `convert_require` points at `[requires]`, `inject_global_value` at `[defines]`, and `remove_spaces` at `generator = "dense"`.

### `Side effects`

```luau
remove_calls = { functions = ["print"], preserve_arguments_side_effects = true }
remove_assertions = { preserve_arguments_side_effects = true }
remove_debug_profiling = { preserve_arguments_side_effects = true }
```

All three take the same table. `remove_calls` is the one people meet first, but `assert(spend())` and `debug.profilebegin(name())` have exactly the same footgun, and `larvae.example.toml` shows both as a bare `= true`.

### `[defines]`

```luau
[defines]
DEBUG = true
VERSION = "1.4.2"
```

Compile-time constants. The name is replaced by the literal wherever it appears as a global, and the folding rules then work on the result.

### `[profile.<name>]`

```luau
[profile.release]
defines = { DEBUG = false }

# larvae process --profile release
```

Nine tables a profile may overlay — `aliases`, `bundle`, `defines`, `fmt`, `lint`, `minify`, `process`, `requires`, `rules` — merged over the root when `--profile` is passed. Tables merge per key; arrays and scalars are replaced whole.

> Root keys, `extends`, `[check]`, `[rojo]`, `[worms.*]` and `[lsp]` are **not** among them: a profile changes how this tree builds, not what the tree is or what the editor answers.

## The gate & CI

The questions only a whole require graph can answer, and the four commands a job runs to ask them.

### `[check]`

```luau
[check]
cycles = "deny"                    # allow | warn | deny — no info
unused_modules = "warn"
early_require = "warn"
entries = ["tools/publish.luau"]
```

Whole-project checks over the require graph, run by `larvae check`: modules that require each other, modules nothing requires (a Script is never one), and — on `roblox-instance` targets — client requires that do not wait for replication. `entries` names the modules only a tool or a runtime lookup reaches, so they are never reported unused.

> Only `deny` fails the run, and there is no `info` level here. Two findings take no level at all — a require nothing can resolve, and a require that crosses a realm — and are reported whatever `[check]` and `[requires]` say. `early_require` warns on an untouched `roblox-instance` project because `indexing_style` defaults to `find_first_child`, which does not wait.

### `[requires] strict`

```luau
[requires]
strict = true
```

Upgrades warn-level require problems — a case mismatch in a path, say — to errors, so `larvae check` fails on them rather than listing them.

> Off by default. Turn it on once a repo's requires are clean; on a repo being migrated it turns the first run into a wall.

### `The four gates`

```luau
larvae worm install                # first: the worms larvae.toml names are not in the checkout
larvae fmt --check                 # non-zero if any file would change
larvae lint                        # non-zero on a deny
larvae check                       # non-zero on a require that cannot work, or a denied cycle
larvae self sync-luaurc --check    # non-zero if .luaurc has drifted from [aliases]
larvae process --profile release   # the build, after every gate
```

What a CI job runs. The four gates write nothing and each fails on its own condition; the build comes last.

> Nothing that writes to the tree belongs in a job — `larvae fmt` without `--check`, or `larvae init` — because a job that changes files is a job whose green means nothing. Pin the release in the tool manager's manifest: a formatter that gains an opinion between two runs changes what `--check` accepts, and a linter that gains a lint turns a job red with nobody's commit to blame. `larvae self update` is for your machine.

## The editor

One `[lsp]` table, and the second binary that carries the types.

### `[lsp] claim_only`

```luau
[lsp]
claim_only = false   # the default
```

`true` answers only for the files a worm claims, so stock luau-lsp keeps every plain `.luau` document. This is the partial-adoption switch: larvae owns your `.luaux` files and nothing else. The plan upstream is to make it the default once the analyzer era settles, as a stated breaking change.

### `[lsp] enabled / analyzer`

```luau
enabled = true
analyzer = true
```

`enabled = false` answers every request with nothing, so another server owns the files entirely. `analyzer = false` keeps the lints, formatting, code actions and outline and drops hover, completion and type diagnostics — the serving larvae had before the analyzer landed.

> `[fmt] enabled = false` and `[lint] enabled = false` are the same idea per tool: take the require rewriter alone and keep stylua and selene.

### `[lsp] roblox_security_level`

```luau
roblox_security_level = "roblox-script-security"
# or "none", "local-user-security", "plugin-security"
```

Which Roblox API surface the analyzer types against. The default holds every member; a plugin or a LocalScript sees less at runtime, and this is how the editor is told so.

### `[lsp] definitions / documentation`

```luau
# definitions = ["types/global.d.luau"]
# documentation = ["types/documentation.json"]
```

Extra type definition files and their docs, the same files `larvae analyze --definitions` takes. `sourcemap` and `sourcemap_autogenerate` sit here too: the server can run rojo's sourcemap watch itself, or stay out of the way of one you already run.

### `[lsp] sourcemap, and the [lsp.*] subtables`

```luau
[lsp]
sourcemap = "sourcemap.json"                  # default
sourcemap_autogenerate = true                 # the server runs the command itself
rojo_project_file = "default.project.json"    # what that command reads

[lsp.completion.imports]
require_style = "auto"     # always_relative | always_absolute | nearest_absolute
use_const = true

[lsp.inlay_hints]
parameter_names = "literals"   # none | literals | all
```

The analyzer types against two things it cannot infer from the source: the shape of the DataModel, which the Rojo sourcemap gives it, and the globals the project defines, which `definitions` files give it. The server can generate the sourcemap itself or stay out of the way of a watch you already run.

> `[lsp.completion]`, `[lsp.completion.imports]`, `[lsp.inlay_hints]`, `[lsp.signature_help]`, `[lsp.hover]`, `[lsp.index]`, `[lsp.studio]` (off by default — it opens a socket, port 3773) and `[lsp.fflags]` mirror luau-lsp's settings under larvae's names. Every inlay hint is off by default. `larvae analyze --definitions <file>` takes the same definitions on the command line, so CI and the editor type against the same world.

## Worms: extensions

How larvae reaches a language that is not Luau.

### `[worms]`

```luau
[worms.luaux]
repo = "larvae-luau/luaux-worm"
version = "^"

[worms.mine]
path = "build/myworm"
```

A worm is a **table**. `repo` requires a `version` — `"^"` for the newest release on every install, a number to hold one, or a range. `cargo = "luaux-worm"` takes it from crates.io instead, which needs no per-platform asset. A `path` runs a local build. The long form adds `asset`, `run_order`, `rules` and a `config` table handed to the worm untouched.

> The key is the repository name with `-worm` removed, which is why `luaux-worm` becomes `luaux`.
>
> Both crate spellings parse: `cargo = "luaux-worm"` beside a separate `version`, and `cargo = "luaux-worm@0.1.0"` with the version inside the string — which excludes `repo`, `path` and `asset`.
>
> The string form `luaux = "owner/repo@version"` was removed in v0.5.0 and is now refused outright, though it still appears in older configs and in larvae's own example file.

### `Fetching one`

```luau
larvae worm add luaux
larvae worm install
```

Two steps, not one. `add` writes the table — `luaux` is a short name larvae knows, so it expands to `larvae-luau/luaux-worm` with `version = "^"` — and stops there, because editing a file is instant and offline while a download is neither. Nothing downloads a worm except `larvae worm install`.

> The editor never downloads either: the LSP skips a worm it does not have, so an editor quietly degrades until someone runs install in a terminal.

### `asset`

```luau
asset = "markup-worm.zip"
# default: <name>-worm-<arch>-<os>.zip,
#          then <name>-worm.zip, then worm.zip
```

Which file to take from the release. The platform name comes first because a native worm ships one artifact per platform; a portable wasm or Luau worm has no such asset and the plain names answer. Get this wrong when publishing and install fails.

### `form`

```luau
form = "luau"   # or "wasm", or "native"
```

Three of them. `luau` runs in an embedded VM and `wasm` in an interpreter; only `native` is an ordinary executable larvae talks to over a pipe. Native has native speed and **no sandbox** — it runs with the user's full access — so it is not the default and a project opts in per worm.

> `[lsp]` hooks need the native form: a manifest that declares them on a luau or wasm worm is refused at load.

### `worm.toml`

```luau
name = "luaux"
api = 1
form = "native"
entry = "luaux-worm"

[frontend]
claims = [".luaux"]
```

A worm's own manifest: what it is called, which API version it speaks, how it runs, and which file extensions it claims.

### `The three jobs`

```luau
compile -> Luau source text
fmt     -> a layout document
lint    -> findings without a severity
```

A worm compiles, formats and lints, but returns less than you might expect for the last two. It hands back a layout document rather than text so larvae applies the project's width and indentation, and findings without levels so larvae stamps them, applies `-- larvae: allow(...)` and decides the exit code.

> The consequence is that a claimed file formats and reports exactly like a Luau file, without the worm owning a printer or a severity scale.

### `[lints.<name>] in worm.toml`

```luau
[lints.compile_error]
description = "…"
default = "deny"
```

A worm declares its lints with bare names and any of the four levels; larvae files them under the worm's key. A project writes `[lint.rules.luaux]`, and everywhere else the lint is `luaux.compile_error` — in a message, in `--explain`, and in an allow comment.

### `A worm's namespace`

```luau
[lint.rules.luaux]              # bare here; luaux.useless_fragment everywhere else
useless_fragment = "deny"

[fmt.luaux]
attribute_per_line = true

[worms.luaux]
run_order = "before"            # or "after", or a slot number
rules = { tidy = true }         # the worm's own rules, by its own names

[worms.luaux.config]            # the worm's own settings, handed over untouched
pretty = true
```

Everything a worm adds is filed under the key it was named by, so a worm's names can never collide with a builtin's: its lints under `[lint.rules.<key>]`, its formatter options under `[fmt.<key>]`, its own rules under `rules`, and its own settings under `[worms.<key>.config]`, which larvae passes on without reading inside.

> `[lint.rules.<key>]` is the only place a worm's lint is written bare; in a finding, in `larvae lint --explain` and in an allow comment it is `<key>.<name>`. `run_order` places the worm's rules among larvae's own, whose slot is `[process] run_order` — its default is `1`, and that number *is* the slot larvae's own rules occupy, so a worm's `"before"` and `"after"` are the slots either side of it and a numeric `run_order` is read on the same scale. The key is the repo name with `-worm` removed, which is what `larvae worm add` writes.

### `inherit_lints / inherit`

```luau
inherit_lints = true
inherit = { lints_except = ["unused_variable"] }
# or lints_only = […], fmt_except = […]
```

Whether larvae's own lints and formatter options run inside this worm's files, and which. `lints_only` and `lints_except` are exclusive — a list of what to keep and a list of what to drop cannot both be the answer — and `fmt_except` names the options that keep their own default there.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [larvae](https://github.com/larvae-luau/larvae) | The toolchain | 0.8.1 | `cargo install larvae larvae self install` |
| [larvae-lsp](https://crates.io/crates/larvae-lsp) | The editor server, with the Luau analyzer | 0.8.1 | `cargo install larvae-lsp` |
| [luaux-worm](https://github.com/larvae-luau/luaux-worm) | A worm for LuauX markup | — | `larvae worm add luaux larvae worm install` |
| [Rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager: pins larvae per repository | 1.2.0 | `rokit add larvae-luau/larvae` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Syncs the derived build project into Studio | 7.7.0 | `rokit add rojo-rbx/rojo` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **Any count of larvae's lints you read outside this page.** The repo README says "thirty four lints", `larvae.example.toml` says sixty-three and gives the split as 6/11/46, and the source has **64** at 6 deny, 11 allow, 47 warn. The README is the one a learner hits first, in its own first screen.
- **`larvae fmt` described as stylua parity out of the box.** `magic_trailing_comma`, `space_inside_braces` and `trailing_comma` are larvae's own and default to on, so the first run will touch your tables. `[fmt] recommended = false` is the switch that makes parity true, and it appears in neither the README nor `larvae.example.toml`.
- **`[worms] name = "owner/repo@version"`, and `larvae worm develop`.** The string form was removed in v0.5.0 and is refused with a message telling you what to write; the `develop` subcommand never existed — it is `larvae worm run`. Both still appear in upstream docs, including `larvae.example.toml` and `luaux-worm`'s README.
- **"Naming a worm makes larvae fetch it."** `larvae.example.toml` says so and it is not true: no command downloads a worm except `larvae worm install`, and the editor never downloads at all.
- **`larvae-lsp` 0.8.0 from crates.io.** It was never published — the build script rewrote a vendored source in place and cargo refused the package. Install 0.8.1, or 0.8.0 from the git tag.
- **A `selene.toml` or `stylua.toml` you were told to delete.** Both are still read, and `[lint]` and `[fmt]` layer over them. Migrating can be gradual.
- **Advice to run Luau's own lints alongside larvae's.** larvae covers all 28, keeping Luau's level for each and reporting the narrower type-free case where Luau would consult the type checker. `larvae init` disables them in `.luaurc` — but only when there is a `.luaurc` that has not already spoken about `lint`, so check yours.
- **"`[check]` is a set of levels you can turn off."** Three of its four keys take a level and `allow` really does silence them — but an unresolvable require and a realm violation are reported whatever `[check]` and `[requires]` say, because they are requires that cannot work rather than opinions about style. `[check]` also has no `info` level, although `[lint]` does.
- **A `.darklua.json` pasted whole into `[rules]`.** Every darklua rule *name* is accepted, which is what a migration guide promises — but three of them are not rules here, and larvae refuses them by name: `convert_require` is `[requires] target` plus `[aliases]`, `inject_global_value` is a line under `[defines]`, and `remove_spaces` is `generator = "dense"`. The generator names carry across with one character changed: `retain_lines` → `retain-lines`.
