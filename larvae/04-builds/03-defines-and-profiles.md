# Defines and profiles

*Compile-time constants that fold away, and one config that builds debug and release.*

Larvae · Builds & Profiles · 11 min

---

`[defines]` are compile-time constants. The name is replaced by its literal wherever it appears as a global, and then the folding rules get to work on what is left — so a `DEBUG` block does not merely evaluate to false, it disappears.

**A define plus folding**

```toml
[defines]
DEBUG = true
VERSION = "1.4.2"

[rules]
compute_expression = true
remove_unused_if_branch = true
```

With `DEBUG = false`, `if DEBUG then … end` becomes a branch with a constant condition, and `remove_unused_if_branch` prunes it entirely.

## Profiles

A profile holds the same keys as the root and merges over it when `--profile <name>` is passed. Tables merge key by key; arrays and scalars are replaced whole. That distinction matters — an array of globs in a profile *replaces* the root's list rather than adding to it.

**Debug by default, release on demand**

```toml
[defines]
DEBUG = true

[process]
output = "dist"

[profile.release]
defines = { DEBUG = false }

[profile.release.process]
output = "dist-release"

[profile.release.rules]
remove_comments = true
rename_variables = true

[profile.release.lint.rules]
unused_variable = "deny"
```

`larvae process --profile release` writes elsewhere, with logging folded out and names shortened.

> **Arrays replace, tables merge**
>
> `[profile.release.process] exclude = ["**/*.spec.luau"]` does not add to the base's `[process] exclude` — it replaces it. Write the full list in the profile, or keep the shared list where the profile does not touch it: the root `exclude` is a different key, inherited by every command, and a profile's `[process]` list leaves it alone.

Profiles are where the day's two builds live. The root is what you run under `--watch` all day, with `DEBUG = true` and comments kept; the release profile is what CI runs once, with logging folded out and, if you want it, the output minified — which is the next lesson. One file, and the difference between the two builds is readable in one place.

**Key points**

- `[defines]` replaces a global name by a literal; `compute_expression` and `remove_unused_if_branch` then delete what became dead.
- `[profile.<name>]` mirrors the root and merges over it on `--profile <name>`.
- Tables merge per key; arrays and scalars are replaced whole — a profile's glob list replaces the root's.
- Root for the debug loop, a profile for the release build, one file for both.
