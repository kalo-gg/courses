# Identity: name, version, includes

*The top-level fields that say who a package is, and the hyphen that silently turns a name into a Wally one.*

pesde · The Manifest · 10 min

---

Every manifest opens with the fields the registry page prints and the publish step validates. Most are what you would guess; two of them — `name` and `includes` — have rules that bite people coming from Wally, and both bite silently.

**The top of a manifest**

```toml
name = "acme/package"
version = "1.2.3"
description = "A package that does foo and bar"
license = "MIT"
authors = ["John Doe <john.doe@acme.local> (https://acme.local)"]
repository = "https://github.com/acme/package"
private = false
includes = ["pesde.toml", "README.md", "LICENSE", "init.luau"]
```

| Field | Notes |
| --- | --- |
| `name` | `scope/name`. Lowercase letters, numbers and underscores only; scope 3–32 characters, name 1–32. Neither half may be all digits, or start or end with `_`. |
| `version` | SemVer. A published version can never be replaced. |
| `license` | An SPDX identifier — `MIT`, `Apache-2.0`. |
| `authors` | `Name <email> (url)`; the email and URL are optional. |
| `private` | `true` refuses to publish. The right setting for a game. |
| `includes` | Globs. **Anything not listed is not published.** |

## Two traps

> **A hyphen silently means Wally**
>
> pesde names use underscores, and a hyphen is not merely invalid — it is the marker for a **Wally** name. Anything containing one is parsed as a Wally package with or without the `wally#` prefix, so `pesde add acme/my-package` never reaches pesde's own validator: it goes looking for a Wally index and fails with `no wally index found`. If you are migrating from Wally, rename `my-package` to `my_package` before you look for the bug anywhere else.

> **The first person to publish a scope owns it**
>
> Scopes are claimed, not registered. To let a teammate publish under yours, send a pull request to `pesde-pkg/index` adding their GitHub user ID to the scope's `scope.toml` — and they must confirm. At least one package has to exist in the scope first.

`private = true` is the right setting for a game and for a workspace root: it refuses to publish, so no whitelist mistake can leak a place file to the registry.

A library cannot use that escape, so its safety net is `pesde publish --dry-run`: it builds the tarball without uploading, and it is the only way to see what `includes` actually caught before consumers do — the listing names every file the tarball would carry, so a missing `src/init.luau` or an accidentally shipped place file is one line to spot.

**The two settings that decide publishability**

```toml
# a library names what ships
includes = ["pesde.toml", "README.md", "LICENSE", "src/**/*.luau", "docs/**/*.md"]

# a game or a workspace root ships nothing, ever
private = true
```

Two different answers, not two lines of one manifest. `includes` is a whitelist — anything unlisted is missing from the tarball; `private = true` refuses to publish before a whitelist mistake can matter.

**Key points**

- `name` is `scope/name`, lowercase with underscores — a hyphen means Wally.
- A published `version` can never be replaced.
- `includes` is a whitelist; `--dry-run` shows what it caught.
- `private = true` for games and workspace roots.
- The first publisher owns the scope.
