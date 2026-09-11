# Your first manifest

*wally init writes four required keys, and every one of them means something you should choose.*

Wally · Wally Foundations · 10 min

---

`wally init` scaffolds a manifest in the current directory. It guesses your scope from your git configuration and the package name from the folder name, so what it writes is a starting point rather than an answer — but it does write every key that is actually required, which is the useful part.

**Scaffolding**

```bash
$ wally init
Initialized project biff/dungeon in /home/biff/dungeon

$ cat wally.toml
[package]
name = "biff/dungeon"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
```

Four keys and an empty table. Everything else in the manifest is optional.

**`name`** is `scope/name`. Each half is lowercase ASCII letters, digits and hyphens, between 1 and 64 characters. There are no uppercase letters and no underscores, so `MyGame` and `drop_table` are both rejected — and if you have come from pesde, note that the hyphen rule is exactly inverted there.

**`version`** is SemVer, and it is the number other people will write requirements against. **`registry`** is the git URL of the index to resolve from; `init` writes the public one. **`realm`** says which of a consumer's three dependency tables is allowed to hold you — Module 2 is largely about that word, and it is the key people get wrong.

## The optional half

The rest of `[package]` matters only when you publish, but it is worth knowing now, because two of the keys change what ends up in the archive and one of them stops you publishing by accident.

**A fuller [package]**

```toml
[package]
name = "biff/drop-table"
version = "1.2.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"
description = "Weighted drop tables for Roblox"
license = "MIT"
authors = ["Biff <biff@example.com>"]
exclude = ["tests/**", "**/*.spec.luau"]
```

`private = true` belongs here too, on anything that is a game rather than a library.

| Key | What it controls |
| --- | --- |
| `description` | Free text shown wherever the package is listed |
| `license` | An SPDX identifier; the client does not validate it |
| `authors` | Free-form strings — nothing parses them |
| `private` | Refuses to publish at all. The correct setting for a game |
| `include` | Globs whose matches are kept in the published archive |
| `exclude` | Globs whose matches are dropped — the half that actually removes files |

`include` and `exclude` are not the symmetric pair they look like; Module 5 shows why.

**Key points**

- `wally init` writes `name`, `version`, `registry` and `realm` — the four keys that are required.
- A name is `scope/name` in lowercase ASCII letters, digits and hyphens, 1–64 characters per half.
- `realm` describes *you*, and constrains which of a consumer's tables may hold you.
- `private = true` is the key that stops a game being published by accident.
- Everything else in `[package]` only matters at publish time.
