# pesde reference

*pesde 0.7.3, released March 18, 2026. Checked on September 7, 2026.*

---

## pesde.toml

Identity, what ships, and what it runs on.

### `name`

```luau
name = "acme/package"
```

`scope/name`. Lowercase letters, numbers and underscores only; the scope is 3–32 characters and the name 1–32. Neither half may be all digits, and neither may start or end with `_`. The first person to publish to a scope owns it.

> A name containing a **hyphen** is parsed as a *Wally* name, with or without the `wally#` prefix — so `pesde add acme/my-package` looks for a Wally package and fails with `no wally index found`, not a name error.

### `version`

```luau
version = "1.2.3"
```

SemVer. A published version can never be replaced.

### `license`

```luau
license = "MIT"
```

An SPDX identifier.

### `authors`

```luau
authors = ["John Doe <j@acme.local> (https://acme.local)"]
```

Name, then an optional email in angle brackets and URL in parentheses.

### `private`

```luau
private = true
```

Refuses to publish. The right setting for a game, and for a workspace root.

### `includes`

```luau
includes = ["pesde.toml", "README.md", "src/**/*.luau"]
```

Globs listing what goes into the published tarball.

> A **whitelist** — anything unlisted is not published, including a `docs/` directory. `pesde publish --dry-run` builds the tarball so you can check.

### `workspace_members`

```luau
workspace_members = ["pkgs/*"]
```

Globs naming the packages of a workspace.

### `description`

```luau
description = "Weighted drop tables for Roblox"
```

Shown on the registry page.

> Versions of one package published for different targets must share it.

### `repository`

```luau
repository = "https://github.com/you/loot"
```

Where the source lives; printed on the registry page.

## [target]

### `environment`

```luau
environment = "luau"  # lune | roblox | roblox_server
```

Which runtime the package is for. Decides the install directory — one per target, so `luau_packages/`, `lune_packages/`, `roblox_packages/` and `roblox_server_packages/`.

> `roblox_server` is **not** a client-size optimization — it complicates linking and is redundant on a public registry. It exists for private registries holding internal code.

### `lib`

```luau
lib = "init.luau"
```

The entry point `require` resolves to. Allowed in every environment.

### `bin`

```luau
bin = "main.luau"
```

The entry point that runs. **Allowed only in `luau` and `lune`** — a Roblox package cannot export one.

### `build_files`

```luau
build_files = ["src"]
```

Roblox targets only, and **required**: the top-level files and directories a sync tool should bring into Roblox. `pesde publish` refuses a Roblox target without it.

> Never list `roblox_packages`; it is always synced. This is what replaces a `default.project.json` — the consumer's `roblox_sync_config_generator` reads it and writes the config their own sync tool needs.

### `[target.scripts]`

```luau
[target.scripts]
roblox_sync_config_generator = "scripts/gen.luau"
```

Scripts a scripts package exports to whoever initializes a project with it. **Allowed only in `luau` and `lune`** — the two targets a scripts package can be built for.

## Dependencies

An alias on the left, a specifier on the right.

### `pesde`

```luau
foo = { name = "acme/foo", version = "1.2.3", index = "acme", target = "lune" }
```

A registry package. `index` defaults to `default`; `target` defaults to your own.

### `Wally`

```luau
foo = { wally = "acme/foo", version = "1.2.3", index = "acme" }
```

A Wally package, for Roblox projects. `index` is resolved against `[wally_indices]`.

### `git`

```luau
foo = { repo = "acme/pkgs", rev = "aeff6", path = "foo" }
```

`repo` is `owner/name` or a full URL; `rev` is a tag, a branch or a commit hash. `path` picks one package out of a repository.

### `workspace`

```luau
foo = { workspace = "acme/foo", version = "^", target = "luau" }
```

A sibling package. Becomes an ordinary pesde dependency at publish time. `version` is optional and defaults to `^`; `target` picks which of the sibling's targets you mean.

### `path`

```luau
foo = { path = "/home/user/foo" }
```

An absolute path to a directory holding a `pesde.toml`.

> **Forbidden in published packages** — it names a directory only your machine has.

### `[dev_dependencies]`

```luau
pesde add --dev acme/spec
```

Not installed for consumers. `pesde install --prod` skips them; `--dev` links only them.

### `[peer_dependencies]`

```luau
pesde add --peer acme/framework
```

Required but not installed — the consumer supplies it, so there is exactly one.

### `[indices]`

```luau
[indices]
default = "https://github.com/pesde-pkg/index"
acme = "https://github.com/acme/pesde-index"
```

Named pesde registries. `[wally_indices]` is the separate table for Wally ones.

### `[wally_indices]`

```luau
[wally_indices]
default = "https://github.com/UpliftGames/wally-index"
```

Named Wally registries; a Wally specifier's `index` resolves here, never in `[indices]`.

> Without a `default` here, `pesde add acme/my-package` (a hyphen) fails with `no wally index found`.

## Workspace version sigils

Three are filled in at publish time; two are left alone.

### `^`

```luau
version = "^"
```

Published as `^2.1.5` when the sibling is at 2.1.5.

### `~`

```luau
version = "~"
```

Published as `~2.1.5`.

### `=`

```luau
version = "="
```

Published as `=2.1.5`.

### `*`

```luau
version = "*"
```

Published unchanged.

### `an explicit range`

```luau
version = "^2.1.0"
```

Published unchanged — you said what you meant.

## Other sections

### `[scripts]`

```luau
[scripts]
build = "scripts/build.luau"
sourcemap_generator = ".pesde/scripts/sourcemap_generator.luau"
```

Scripts run by `pesde run`, executed with **Lune**.

> Two are special: `sourcemap_generator` and `roblox_sync_config_generator`.
>
> `sourcemap_generator` receives the package directory as the first `process.args`, and its sourcemap is how pesde finds a **Wally** dependency's entry point at all — a Wally package has no pesde manifest declaring a `lib`. Without the script the linker points at `____pesde_no_export_file_found`, so the dependency has no working entry point, not merely no types.
>
> A scripts package links its scripts under `.pesde/<alias>/<name>.luau`, where the alias is the dev-dependency's — so `pesde init` writes `.pesde/scripts/…`, not `.pesde/…`. Get the path wrong and nothing errors; the script simply never runs.

### `[overrides]`

```luau
[overrides]
"foo>bar" = { name = "acme/bar", version = "^2.0.0" }
"foo>baz,qux>baz" = "baz"
```

Replace a transitive dependency. The key is a `>`-separated path of aliases; several paths can share a rule with commas.

> The value may be a specifier, or an **alias** of one of your own dependencies, meaning "use mine".

### `[patches]`

```luau
[patches]
"acme/foo" = { "1.0.0 luau" = "patches/acme+foo-1.0.0+luau.patch" }
```

Written by `pesde patch-commit`. Keyed by package, then by `<version> <target>`.

> Mark `*.patch binary` in `.gitattributes` so line endings survive, and never stage the temporary patch directory.

### `[place]`

```luau
[place]
shared = "game.ReplicatedStorage.Packages"
server = "game.ServerScriptService.Packages"
```

Where installed packages sit in the Roblox datamodel, so requires resolve inside a running game.

> Read only when a linker crosses between targets' packages directories (`roblox_server` → `roblox` or back); a project's own direct dependencies never read it, whatever their origin, and 0.7.2's string requires did not change that.

### `[engines]`

```luau
[engines]
pesde = "^0.6.0"
lune = "^0.8.9"
```

Runtime versions this package is compatible with. Only `pesde` and `lune` can be named; `pesde install` sets them up and consumers are warned on a mismatch.

> `init` copies the scripts package's engines into a new project.

## Setup

The binary, its home, and what a project starts with.

### `pesde self-install`

```luau
pesde self-install
pesde self-upgrade --use-cached
```

The first command to run after downloading the binary. `self-upgrade` updates it.

> `self-install --skip-add-to-path` (Windows only) leaves `PATH` alone. `self-upgrade --use-cached` takes the version from the "upgrades available" message instead of checking again; `--include-pre` allows pre-releases.

### `PESDE_HOME`

```luau
PESDE_HOME=/opt/pesde pesde install
```

Environment variable that moves pesde's home — the binary, the store and its config — from `$HOME/.pesde`.

> Cache it between CI runs and the store stays warm.

### `pesde config default-index`

```luau
pesde config default-index
pesde config default-index https://github.com/acme/pesde-index
pesde config default-index --reset
```

Read or set the index `pesde init` scaffolds with and `pesde x` fetches from, machine-wide. With no argument it prints the current one; `-r, --reset` restores the built-in default.

### `pesde cas`

```luau
pesde cas path
pesde cas prune
```

`path` prints the content-addressable store this location uses; `prune` drops files and packages nothing references.

### `pesde init`

```luau
pesde init
```

Scaffold a project, choosing an environment and optionally a Roblox scripts package.

### `pesde.lock`

```luau
pesde.lock  # generated
```

Written by `pesde install`; records each dependency's exact version and target and the graph between them, with a generated-file marker.

> Commit it. `add` never touches it; `update --no-install` touches only it; `install --locked` rewrites it byte-identically and refuses to re-resolve a stale one; `publish` refuses a stale one for `roblox` and `roblox_server` targets.

### `aliases`

```luau
pesde add pesde/hello --alias greet
```

The key on the left of a dependency entry: what you require and what `remove` takes. Matches `[a-zA-Z0-9_-]+` and is at most 48 characters.

> `self`, `cart`, the engine names (`pesde`, `lune`) and Windows' reserved file names are refused.

### `linkers`

```luau
luau_packages/<alias>.luau
```

The file `install` writes per alias in the target's packages directory; it requires the package in the store and re-exports its types (type functions included since 0.7.3).

> Require the linker by path or alias; never reach into `.pesde/`.

## CLI

The per-project commands.

### `pesde add`

```luau
pesde add pesde/hello@1.2.3
pesde add wally#acme/bar
pesde add gh#acme/pkg#main
pesde add workspace:acme/qux
pesde add path:/home/user/foo
```

Add a dependency. `--peer`, `--dev`, `--alias`, `--index`, `--target`.

### `pesde install`

```luau
pesde install --locked
pesde i   # the same command
```

Resolve, download and link.

> `--locked` fails on a stale lockfile — the CI form. Also `--prod`, `--force`, `--network-concurrency` (default 16), and `--dev`, which links only dev dependencies (0.7.0).

### `pesde update`

```luau
pesde update --no-install
```

Re-resolve. `--no-install` touches only the lockfile.

### `pesde remove`

```luau
pesde remove <alias>
```

Takes the **alias**, not the package name.

### `pesde list / outdated`

```luau
pesde outdated --strict
```

What is installed, and what has moved on. `-s, --strict` checks only within your declared version requirements.

### `pesde run`

```luau
pesde run build -- --arg1
```

Resolves in order: no argument → your own `target.bin`; then a **package name or alias** → its `target.bin`; then a `[scripts]` name → that script; then a **path** → that file.

> The package lookup comes first, so a dependency aliased the same as one of your scripts shadows it. The prose docs list the reverse order; `run.rs` is the authority.

### `pesde x`

```luau
pesde x pesde/hello
pesde x -t lune acme/build_tool
pesde execute … # or `pesde exec`
```

Run a binary package once, without a project. `_G.PESDE_ROOT` is then the temporary directory it was downloaded into.

> `-t, --target` **defaults to `luau`**, so a package published only for Lune needs `-t lune` or it will not resolve. A Roblox target is rejected outright: `roblox doesn't support bin exports!`. `-i, --index` names the registry URL to fetch from, otherwise your configured default index is used.

### `pesde publish`

```luau
pesde publish --dry-run
```

Publish. `-y` skips the prompt, `-i` picks an index, `--no-verify` skips export syntax validation.

### `pesde yank`

```luau
pesde yank "acme/foo@1.0.0 luau"
pesde yank acme/foo@1.0.0   # all targets
```

Blocks fresh installs of a version. Lockfiles that already have it keep working.

> The argument is `scope/name@version` with an optional target after a space — **omit the target and every target of that version is yanked**. `--undo` reverses it.

### `pesde deprecate`

```luau
pesde deprecate acme/foo "superseded by acme/bar"
```

Warns on the whole package. A non-empty reason is required.

### `pesde patch`

```luau
pesde patch "acme/foo@1.0.0 luau"
pesde patch-commit /x/y/z
```

Copy a dependency's source out, edit it, and turn the edits into a patch.

### `pesde auth`

```luau
pesde auth login
pesde auth -i acme login --token ghp_…
```

GitHub device-code flow. Also `logout`, `whoami`, `token`.

> `-i, --index` sits on the **group**, before the subcommand, and picks which index the token is for. `login -t, --token` stores a token directly and skips the browser. `token` prints the stored token, which is how a CI secret is made.

### `scope.toml`

```luau
owners = [1234567, 7654321]
```

The ownership record for a scope, in `pesde-pkg/index`; GitHub user IDs, the true owner first. Team members are added by pull request and must confirm.

> `owners` is the only field the file has.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [pesde](https://github.com/pesde-pkg/pesde) | The package manager | 0.7.3 | `download a release, then `pesde self-install`` |
| [pesde-pkg/index](https://github.com/pesde-pkg/index) | The default registry index | registry 0.2.3 | `[indices] default = "https://github.com/pesde-pkg/index"` |
| [pesde-pkg/scripts](https://github.com/pesde-pkg/scripts) | Sync config and sourcemap generators | — | `pesde init, then pick a scripts package` |
| [Rojo (`pesde/rojo`)](https://github.com/rojo-rbx/rojo) | The sync tool the course's scripts package targets | 7.5.x | `pesde add --dev --target lune pesde/rojo` |
| [Lune](https://lune-org.github.io/docs) | The runtime `[scripts]` and Lune-target binaries run on | 0.10.x | `[engines] lune = "^0.10.2"` |
| [Wally index (`UpliftGames/wally-index`)](https://github.com/UpliftGames/wally-index) | The registry Wally specifiers resolve against | — | `[wally_indices] default = "https://github.com/UpliftGames/wally-index"` |
| [Luau LSP](https://github.com/JohnnyMorganz/luau-lsp) | Editor types across your package and its dependencies | — | `set `luau-lsp.sourcemap.rojoProjectFile` to `test-place.project.json`` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`default.project.json` in a package.** That is Wally's convention, and `pesde publish` refuses a package containing one. You declare the 1:1 mapping with `build_files` instead, and the consumer's `roblox_sync_config_generator` writes the project file for them — keep a separate `test-place.project.json` for your own Studio testing.
- **Installing pesde with a toolchain manager.** Rokit and Aftman are explicitly discouraged for pesde itself. Use `pesde self-install` and `pesde self-upgrade`.
- **Expecting to delete or replace a published version.** Neither is possible. `pesde yank` blocks fresh installs of one version — one target of it, or every target if you omit the target; `pesde deprecate` warns on the whole package. Removal is reserved for policy violations, security issues and legal requirements.
- **Reading pesde's `main` branch as current — and `build_files` above all.** The published docs site tracks `main`, and `main` has already *removed* `build_files` and rewritten the pages that describe it. In 0.7.3 it is **required**: `pesde publish` refuses a Roblox target whose `build_files` is missing or empty. Everything else on the docs site is byte-identical between 0.7.3 and `main`, so this is the single field where the live docs hand a 0.7.3 user an unpublishable manifest.
- **`main`'s other unreleased changes, none of which have reached the docs yet.** Scripts packages, workspace packages and toolchain management are all removed; `[place]` is renamed `absolute_paths` and repurposed for string requires; string requires become universal, including for Wally packages, which undercuts the `[place]`-and-sourcemap story in Roblox Projects; patches move to the workspace root and must be declared there rather than in a member's manifest; workspaces stop being a "run on" wrapper and are integrated properly; peer dependencies are always rechecked even when the lockfile is reused; and the CAS is restructured with versioning, so you are advised to delete yours before upgrading. Also coming: checksums on packages, a toggle for global binary linkers, decentralised packages, and `init` reserved as an alias. None of this is in 0.7.3.
- **`.pesde/scripts_rojo/…` in the scripts-packages guide.** Scripts link at `.pesde/<alias>/<name>.luau`, and `pesde init` adds the package under the alias `scripts`, so the paths it writes are `.pesde/scripts/…`. Copy the guide's path into a project scaffolded by `init` and the generator silently never runs.
- **`pesde x pesde/hello` printing `(pesde/hello@1.0.0, lune)`.** The binary-packages guide shows a Lune result; the quickstart shows `luau`. `x` resolves the `luau` target unless `-t lune` says otherwise, and a Lune-only package does not resolve at all without it.
- **The CLI reference's `pesde run` order.** The prose lists path, then `[scripts]`, then package; `run.rs` tries a package name or alias first, then `[scripts]`, then a path — so a dependency aliased like a script shadows the script.
