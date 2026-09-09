# larvae.toml

*One file, layered, with short forms for the three keys every project sets.*

Larvae · Larvae Foundations · 11 min

---

Nothing in `larvae.toml` is required. `larvae fmt` and `larvae lint` work with no config at all, and `larvae process` needs at most two sections. What follows is the shape, not a list of obligations.

**A small, complete config**

```toml
input = "src"
output = "dist"
target = "roblox-string"

[aliases]
pkg = "@game/ReplicatedStorage/Packages"

[lint]
std = "roblox"
```

`input`, `output` and `target` are short forms for `[process] input`, `[process] output` and `[requires] target`. Write one spelling — both at once is an error.

> **Root keys come first**
>
> TOML only reads a root-level key before the first table header. If you put `input = "src"` after `[aliases]`, it becomes `aliases.input` and larvae never sees it. This is a TOML rule rather than a larvae one, and it catches everybody once.

## Layering

Three mechanisms stack, which is what lets a monorepo keep its shared settings in one place. `extends` names a base config by path; the base loads first and this file wins key by key. An existing `stylua.toml` or `selene.toml` is read before `[fmt]` and `[lint]`, which layer over it. And `[profile.<name>]` mirrors the root and merges over it when `--profile <name>` is passed — tables merge per key, arrays and scalars are replaced whole. Profiles get a lesson of their own in Builds & Profiles.

| Layer | Loads | Wins |
| --- | --- | --- |
| `extends` base | First | This file, key by key |
| `stylua.toml` / `selene.toml` | Before `[fmt]` / `[lint]` | The tables in this file |
| `[profile.<name>]` | Only with `--profile <name>` | The profile: tables per key, arrays and scalars whole |

**A base config, extended**

```toml
extends = "./shared/larvae.base.toml"

[lint.rules]
shadowing = "deny"     # the base said "warn"; this file wins
```

`extends` is a path relative to *this* file. The base loads first, so a monorepo keeps its shared twenty lines once.

## Excluding things

The first thing a selene or stylua user looks for, because both of their existing configs already exclude `Packages/`. larvae has four levels of it, and one non-obvious rule about how they interact.

**Four places, one shape**

```toml
exclude = ["Packages"]              # root: every command inherits this
include = ["Packages/keep.luau"]   # cancels the root exclude, and only that

[process]
exclude = ["**/*.spec.luau"]

[fmt]
exclude = ["**/*.gen.luau"]

[lint]
exclude = ["Packages", "**/*.spec.luau"]
```

Every list is globs relative to the project root, and a match on any directory on the way to a file counts — so a bare `"Packages"` is enough, with or without a trailing wildcard.

> **The include of one area is stronger**
>
> The root `include` cancels the root `exclude`, and nothing else. An `include` written inside `[fmt]` or `[lint]` cancels that area's own exclude **and** the root one, because an area that names a file has spoken about it more specifically than the root did.
>
> Neither list applies to a file you name on the command line. A formatter that silently does nothing to a file you pointed at is worse than one that formats an excluded file.

**Key points**

- Nothing in `larvae.toml` is required; `input`, `output` and `target` are the root short forms of the three keys most projects set.
- Root keys go before the first table header — TOML files them under it otherwise.
- `extends` loads a base first; existing stylua and selene files load before `[fmt]` and `[lint]`; a profile merges over everything on `--profile`.
- Four exclude lists: the root pair every command inherits, and one each for `[process]`, `[fmt]` and `[lint]`.
