# Overrides and patches

*When a dependency's dependency is wrong you override it; when the dependency itself is, you patch it.*

pesde · Workspaces, Tooling & CI · 10 min

---

You depend on `acme/foo`, which depends on `bar` 1.0.0, and you need `bar` 2.0.0. You cannot edit `foo`'s manifest — so you override it from yours.

**An override**

```toml
[dependencies]
foo = { name = "acme/foo", version = "^1.0.0" }

[overrides]
"foo>bar" = { name = "acme/bar", version = "^2.0.0" }
```

The key is a path of **aliases** separated by `>`, so `foo>bar>baz` is baz inside bar inside foo.

If you already depend on the same thing yourself, the value can be an **alias** instead of a whole specifier, and means "use my one". Several paths can share a rule by separating them with commas.

**Two shorthands**

```toml
[dependencies]
foo = { name = "acme/foo", version = "^1.0.0" }
bar = { name = "acme/bar", version = "^2.0.0" }

[overrides]
"foo>bar" = "bar"                                    # use my bar
"foo>baz,qux>baz" = { name = "acme/baz", version = "1.0.0" }
```

## Patching

An override swaps a package. A **patch** changes one. `pesde patch` copies a dependency's source into a temporary directory; you edit it there, and `pesde patch-commit` turns your edits into a patch file the project applies on every install — a file you commit, and mark `*.patch binary` in `.gitattributes` so git never rewrites its line endings.

**The two-step**

```bash
pesde patch "acme/foo@1.0.0 luau"
# done! modify the files in the directory, then run
# `pesde patch-commit /x/y/z` to apply.

# ... edit files in /x/y/z ...

pesde patch-commit /x/y/z
```

The package is named `<name>@<version> <target>` — the target is part of the identity.

**What lands in the manifest**

```toml
[patches]
"acme/foo" = { "1.0.0 luau" = "patches/acme+foo-1.0.0+luau.patch" }
```

> **Do not stage or commit the temporary directory**
>
> pesde diffs it to build the patch, and staged changes confuse that. Also mark patch files as binary in `.gitattributes` — `*.patch binary` — so git never rewrites their line endings, which produces cryptic failures when the patch is applied.

**Key points**

- `[overrides]` keys are `>`-separated alias paths; commas share one rule across paths.
- The value is a specifier, or the alias of one of your own dependencies.
- `pesde patch` copies the source out; `pesde patch-commit` writes `patches/…` and `[patches]`.
- Never stage the temporary directory; mark `*.patch binary`.
