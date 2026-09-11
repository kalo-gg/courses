# Wally reference

*Wally 0.3.2, released June 5, 2023. Checked on September 9, 2026.*

---

## wally.toml

Who you are, what you ship, and where your packages sit in the datamodel.

### `[package]`

```luau
[package]
```

The only required table. `name`, `version`, `registry` and `realm` must all be present; everything else in it is optional. `wally init` writes exactly those four.

### `name`

```luau
name = "biff/drop-table"
```

`scope/name`. Each half is lowercase ASCII letters, digits and hyphens, 1–64 characters. The scope is claimed by whoever publishes to it first.

> `MyGame` is rejected: there are no uppercase letters and no underscores in a Wally name. A hyphen is fine, which is the opposite of pesde's rule.

### `version`

```luau
version = "0.1.0"
```

SemVer. Published versions are immutable — there is no replacing one and no deleting one.

### `registry`

```luau
registry = "https://github.com/UpliftGames/wally-index"
```

The index this project resolves against. `wally init` writes the public one.

### `realm`

```luau
realm = "shared"  # shared | server | dev
```

Which of a consumer's dependency tables is allowed to hold you. It does **not** decide where your own dependencies land.

> The rule that catches people: the *table* constrains. `[dependencies]` accepts only packages whose manifest says `realm = "shared"`; the other two tables accept any realm.

### `description`

```luau
description = "Weighted drop tables"
```

Free text, shown wherever the package is listed.

### `license`

```luau
license = "MIT"
```

An SPDX identifier. Optional, and not validated by the client.

### `authors`

```luau
authors = ["Biff <biff@example.com>"]
```

Free-form strings. Nothing parses them.

### `private`

```luau
private = true
```

Refuses to publish. The right setting for a game or an app, which has no business in a registry.

### `include`

```luau
include = ["src/**", "wally.toml"]
```

Globs. A file matching one is kept in the published archive.

> **Not a whitelist**, whatever the manifest's own doc comment says. A file is kept if `include` matches it, *and otherwise kept unless `exclude` matches it* — so a short `include` list shrinks nothing.

### `exclude`

```luau
exclude = ["**/*.spec.luau", "tests/**"]
```

Globs. The half of the pair that actually removes files, because it is the only one that can.

> `wally package --list` prints the archive without sending it. Use it before every first publish.

### `[place]`

```luau
[place]
shared-packages = "game.ReplicatedStorage.Packages"
server-packages = "game.ServerScriptService.ServerPackages"
```

Where the installed directories end up in the datamodel. Wally interpolates these two strings into link modules as raw Luau when a link has to cross realms.

> Needed only when something in `ServerPackages/` depends on something in `Packages/`. Omit it and that link cannot be written.

## Dependency tables

An alias on the left, a package spec on the right.

### `[dependencies]`

```luau
[dependencies]
Signal = "sleitnick/signal@^2.0.0"
```

Installs into `Packages/`. Every entry must name a package whose own manifest declares `realm = "shared"`.

### `[server-dependencies]`

```luau
[server-dependencies]
ProfileStore = "biff/profile-store@^1.0.0"
```

Installs into `ServerPackages/`. Accepts a package of any realm.

### `[dev-dependencies]`

```luau
[dev-dependencies]
TestEZ = "roblox/testez@^0.4.1"
```

Installs into `DevPackages/`. Accepts any realm, and is not installed for anyone who depends on you.

> A dependency's own dev-dependencies are never traversed, so a test runner in a library you depend on costs you nothing.

### `The alias`

```luau
Signal = "sleitnick/signal@^2.0.0"
```

The key is free-form and becomes the name of the link module on disk: this one is required as `Packages.Signal`. It has nothing to do with the package's own name.

### `The spec`

```luau
scope/name@version-requirement
```

Three parts, all required.

> The version half is **not** optional. `Signal = "sleitnick/signal"` is a parse error, deliberately: a missing requirement must never become a silent wildcard.

## Version requirements

Cargo's semver crate, with two behaviours that will surprise you.

### `Bare version`

```luau
"biff/minimal@1.2.0"
```

A bare version is a **caret** requirement: `1.2.0` means `^1.2.0`, not `=1.2.0`.

> This is the single most common misreading of a Wally manifest. If you meant exactly one version, say `=1.2.0`.

### `Caret`

```luau
"@^1.2.3"
```

Any version that does not change the leftmost non-zero part: `>=1.2.3, <2.0.0`. For `^0.2.3` that is `>=0.2.3, <0.3.0`.

### `Exact`

```luau
"@=1.2.3"
```

That version and nothing else. The requirement to reach for when a break would be expensive.

### `Tilde`

```luau
"@~1.2.3"
```

Patch-level changes only: `>=1.2.3, <1.3.0`.

### `Comparison`

```luau
"@>=1.2.0"
```

`>`, `>=`, `<`, `<=` all work on their own.

### `Conjunction`

```luau
"@>=1.2.0, <1.5.0"
```

Comma-separated requirements are ANDed. Every one of them must hold.

### `Wildcard`

```luau
"@*"
```

Any version at all. Legal, and almost always a mistake in a manifest you intend to keep.

### `Prereleases`

```luau
"@4.0.0-rc.2"
```

A prerelease matches only a requirement that mentions a prerelease.

> `@4.0.0` does **not** accept `4.0.0-rc.2`. You have to ask for the prerelease by name.

### `Normalisation`

```luau
^1.2.3  →  >=1.2.3, <2.0.0
```

What you wrote is not what is stored. The index records the expanded form, so an index entry never shows you a caret.

## The CLI

Nine subcommands, one of which does not work.

### `wally init`

```luau
wally init
```

Writes a `wally.toml` with the four required keys, guessing `name` from your git config and the folder name. Prints `Initialized project <scope>/<name> in <path>`.

### `wally install`

```luau
wally install [--project-path <dir>]
```

Resolves the graph, writes `wally.lock`, and builds `Packages/`, `ServerPackages/` and `DevPackages/`.

> `--project-path` is its **only** flag in 0.3.2. `--locked` is documented on the default branch and does not exist in the release; the README at the v0.3.2 tag says so itself.

### `wally publish`

```luau
wally publish [--project-path <dir>]
```

Packs the project and uploads it to the registry's publish endpoint.

> **A failed publish exits 0.** It prints `Error: <status>` and then returns success, so CI that trusts the exit code passes on a failed publish. `--token` is on the default branch only.

### `wally login`

```luau
wally login
```

GitHub device-code flow: it prints a code, you paste it into GitHub, and the token is written to Wally's auth file.

### `wally logout`

```luau
wally logout
```

Removes the stored token.

### `wally search`

```luau
wally search <query>
```

Queries the registry's search endpoint and prints matching packages.

### `wally package`

```luau
wally package --list
```

Builds the archive that `publish` would send. `--list` prints its contents instead of writing it; `--output <file>` writes the zip.

> The only honest way to see what `include` and `exclude` actually did.

### `wally manifest-to-json`

```luau
wally manifest-to-json
```

Prints the parsed manifest as JSON. Written for tooling; useful for checking how a requirement was parsed.

### `wally update`

```luau
wally update  # panics
```

Documented, and not implemented.

> **The subcommand's body is a `todo!()` in 0.3.2, so it panics.** It is implemented on the unreleased `main`. To upgrade in the release: edit the requirement, delete `wally.lock`, run `wally install`.

### `wally --version`

```luau
wally --version
```

Prints `wally 0.3.2`. The smoke test after installing.

## What lands on disk

Three directories of link modules over one flat index.

### `Packages/`

```luau
Packages/
```

Shared-realm packages: everything from `[dependencies]`, plus the shared packages they pull in.

### `ServerPackages/`

```luau
ServerPackages/
```

Everything from `[server-dependencies]`, and their non-shared transitive dependencies.

### `DevPackages/`

```luau
DevPackages/
```

Everything from `[dev-dependencies]`. Never installed for a consumer of your package.

### `_Index`

```luau
Packages/_Index/biff_minimal@0.1.0/minimal/
```

The real contents, one flat directory per resolved package: `<scope>_<name>@<version>/<name>/`. Note the underscore between scope and name, and that the inner directory is the short name.

### `Link modules`

```luau
return require(script.Parent._Index["biff_minimal@0.1.0"]["minimal"])
```

One file per alias, at the top of a package directory. That single line is the whole file.

> **Link files are written as `.lua`, not `.luau`** — a long-open pull request. It matters for editors, and it is why a bare `return require(...)` erases the package's exported types.

### `wally.lock`

```luau
[[package]]
name = "biff/one-dependency"
version = "0.1.0"
dependencies = [["Minimal", "biff/minimal@0.1.0"]]
```

The resolved graph, one `[[package]]` per resolved package, with dependencies as `[alias, spec]` pairs.

> `registry = "test"` is hardcoded in the writer, so **every** real lockfile says it. There are no checksums: the field exists in the type and is never filled. The README's lockfile example matches nothing Wally writes.

### `Install is destructive`

```luau
wally install  # deletes Packages/ first
```

`install` removes `Packages/`, `ServerPackages/` and `DevPackages/` **entirely** before writing them.

> Anything you put in there by hand is gone, and anything a tool wrote there — `wally-package-types` above all — has to be re-run afterwards.

## The registry

A git repository of manifests, and an HTTP service in front of it.

### `The index`

```luau
wally-index/bi/ff/biff-package
```

A plain git repository anyone can clone. One file per package, holding append-only newline-delimited JSON — one serialised manifest per published version, oldest first.

### `config.json`

```luau
{ "api": "https://api.wally.run", "github_oauth_id": "…" }
```

At the root of the index: where the client should send API calls, and which GitHub app to authenticate against.

> `fallback_registries` is declared here too — in the *index*, never in your `wally.toml`.

### `owners.json`

```luau
[ 1234567 ]
```

One per scope, holding numeric GitHub user ids. Publishing to an unclaimed scope whose name matches your login claims it and writes this file.

> Add ids to share maintenance. There is no web UI for it.

### `The publish endpoint`

```luau
POST /v1/publish
```

Takes the zip archive with your token in the header. The registry validates the manifest, checks scope ownership and appends to the index.

### `Size cap`

```luau
2 MiB
```

The archive limit. Generous for source, but reachable once art or `.rbxm` files are in the tree.

### `Immutability`

```luau
package already exists in index
```

Republishing a version fails with that message. There is no delete and no overwrite.

> No `wally yank` command exists in 0.3.2, even though the registry's policy page talks about yanking.

## Ecosystem and interop

Wally produces a tree; something else has to put it in a place.

### `Rojo wiring`

```luau
"ReplicatedStorage": { "Packages": { "$path": "Packages" } }
```

`default.project.json` maps each installed directory into the datamodel. The paths you choose here are the ones `[place]` has to agree with.

### `A package is a project`

```luau
default.project.json (inside a package)
```

A directory containing a `default.project.json` is described by that file, which is how a published package becomes a single ModuleScript pointing at its own entry point.

> `wally publish` rewrites that file's `name` to the package's short name, so what you set locally is not what consumers see.

### `wally-package-types`

```luau
wally-package-types --sourcemap sourcemap.json Packages/
```

Rewrites the link modules so exported types survive the indirection. Run `wally install`, then `rojo sourcemap`, then this — in that order.

> Every `wally install` wipes the directory, so this is not a one-off: it belongs in the same script as the install.

### `Realm errors`

```luau
No packages were found that matched (Shared) biff/server@>=1.0.0, <2.0.0.
```

What a realm mistake looks like. The candidate is filtered out of the pool before matching, so the error reads as a missing package.

> The second line — `Are you sure this is a Shared dependency?` — is the real symptom. Move the entry to `[server-dependencies]`.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Wally](https://github.com/UpliftGames/wally) | The package manager this course teaches | 0.3.2 | `rokit add UpliftGames/wally@0.3.2` |
| [Rokit](https://github.com/rojo-rbx/rokit) | The toolchain manager that installs the pinned CLI | 1.2.0 | `rokit init, then rokit add` |
| [Wally index](https://github.com/UpliftGames/wally-index) | The default registry | — | `registry = "https://github.com/UpliftGames/wally-index"` |
| [wally-package-types](https://github.com/JohnnyMorganz/wally-package-types) | Puts exported Luau types back into the link modules | 1.6.2 | `rokit add JohnnyMorganz/wally-package-types` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Syncs the installed tree into a place | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [pesde](https://github.com/pesde-pkg/pesde) | The other Luau package manager, which can read Wally's registry | 0.7.3 | `[wally_indices] default = "https://github.com/UpliftGames/wally-index"` |
| [Foreman](https://github.com/Roblox/foreman) | Roblox's own toolchain manager, recommended by the v0.3.2 README | 1.7.0 | `wally = { source = "UpliftGames/wally", version = "0.3.2" }` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`wally update`, on the default branch's README.** It is documented there as a working command. In the v0.3.2 release the subcommand's body is a `todo!()`, so running it panics. The implementation exists on the unreleased `main`. Until a release carries it, upgrading means editing the requirement, deleting `wally.lock` and running `wally install`.
- **`wally install --locked`, on the default branch's README.** The flag does not exist in 0.3.2 — `--project-path` is `install`'s only flag there. This one is worth checking for yourself: the README **at the v0.3.2 tag** still carries the line saying locked is a planned feature and not yet implemented, and the default branch dropped that disclaimer without shipping the flag.
- **`wally publish --token`, on the default branch's README.** Absent from the release. In 0.3.2 the token comes from `wally login` or the auth file, and there is no way to pass one on the command line.
- **`homepage` and `repository` in the default branch's manifest example.** Neither key is in 0.3.2's manifest schema. The live registry does store them for packages published by newer tooling, so you will see them on package pages — but adding them to a manifest you publish with 0.3.2 does nothing.
- **The README's `wally.lock` example.** It shows `registry:` prefixes, flat dependency strings and checksums. Wally writes none of those: a real lockfile says `registry = "test"`, lists dependencies as `[alias, spec]` pairs, and has no checksums at all.
- **`include` described as a whitelist**, including in the manifest's own doc comment. The filter keeps a file if `include` matches it *and otherwise keeps it unless `exclude` matches*, so a short `include` list does not shrink the archive. Check with `wally package --list` before publishing.
- **Aftman, recommended by wally.run's install page and by the current README.** Aftman is archived and says so on its own first line. The README at the v0.3.2 tag recommends Foreman instead, which is Roblox's and has turned inward. Use Rokit, which is a drop-in for both.
- **Older DevForum guides installing Foreman and depending on `roblox/roact`.** Both were right at the time. The ecosystem has moved to Rokit for toolchains and to `jsdotlua/react` for the React lineage; the Roact packages are still in the index because nothing is ever removed from it.
