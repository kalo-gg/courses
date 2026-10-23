# Installing larvae

*One binary by any route, luau-lsp beside it for the types, and three switches that adopt only part of it.*

Larvae · Larvae Foundations · 10 min

---

larvae is one Rust binary with no runtime dependencies. The CLI and the editor server are the same file: `larvae lsp` is a subcommand, not a second download. Take it from crates.io or from a release zip, then let `larvae self install` put it on your `PATH`. The types are a separate install, and the one people forget.

**Install**

```bash
cargo install larvae

# or unzip larvae-<os>-<arch>.zip from the GitHub release, then:
larvae self install      # copies the binary to ~/.larvae/bin and prints the PATH line
```

> **The types are a second install**
>
> larvae serves no hover, no completion and no type diagnostics. In VS Code, install the **luau-lsp** extension beside the Larvae extension; in Zed, the Luau extension beside larvae's. Skip it and you get lints and formatting in an editor that shows no types, with nothing on screen saying why.

## Through a tool manager

**Pinning it, and the rest of `self`**

```bash
rokit add larvae-luau/larvae      # pins larvae in rokit.toml for everyone who clones
larvae self update                # updates only a copy that self install made
larvae self code                  # completion and hover on larvae.toml itself
larvae self sync-luaurc           # write [aliases] into .luaurc, as paths an editor can follow
larvae self sync-luaurc --check   # write nothing; exit non-zero if it would change
```

A release zip holds the one binary, so a tool manager installs all of larvae. `self update` refuses a copy that rokit, aftman, lpm or cargo owns and points at that manifest instead: bump the pin there, so the manifest and the file on disk agree.

> **Recognising older code**
>
> Guides written before 0.10 install a second crate with `cargo install larvae-lsp` and type-check with `larvae analyze`. Both are gone: the crate stops at 0.9.0 on crates.io, and the subcommand no longer exists. Skip the first. For the second, use `luau-lsp analyze`.

Whichever route you took, the test is the same. Open a Luau file and hover a local: no type means luau-lsp is missing. Then run *Format Document*: nothing happening means the Larvae extension cannot find `larvae`. It looks on `PATH` first, then in `~/.larvae/bin`, and its `larvae.path` setting names any other location.

## Taking only the part you want

You do not have to adopt all of it. Three switches let larvae own one job and stay out of the way on the others — which is worth knowing before you run anything, because "try the require rewriter, keep stylua" is a perfectly good first step.

**Three ways to say "not this part"**

```toml
[fmt]
enabled = false     # keep stylua; larvae never rewrites a file

[lint]
enabled = false     # keep selene

[lsp]
claim_only = true   # the editor server lints and formats only worm-claimed files,
                    # so another linter and formatter keep plain .luau
```

`enabled = false` says it in the config rather than by not running the command — which matters, because the editor reads the same config and would otherwise keep formatting on save. `claim_only` keeps larvae to the files only it can read.

**Key points**

- `cargo install larvae` or a release zip, then `larvae self install`: one binary, with `larvae lsp` inside it.
- luau-lsp is the second install, and it carries the types; `self install` and `self update` delete a leftover `larvae-lsp`.
- `larvae self code` gives your editor completion and hover on `larvae.toml` itself.
- `[fmt] enabled = false`, `[lint] enabled = false` and `[lsp] claim_only = true` let larvae take one job and leave the rest.
