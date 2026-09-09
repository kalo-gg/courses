# Installing larvae

*Two binaries, one of which you can miss without noticing, and three switches that adopt only part of it.*

Larvae · Larvae Foundations · 10 min

---

larvae is a Rust binary, so the direct route is cargo. But "one binary" has the footnote from the last lesson: the Luau analyzer — hover, completion and type diagnostics — lives in a second crate, `larvae-lsp`, and nothing on screen tells you when it is missing. Install both, whichever route you take.

**Install**

```bash
cargo install larvae

# the Luau analyzer lives in a second crate
cargo install larvae-lsp
```

> **Why two crates, and what you lose with one**
>
> `larvae lsp` — the subcommand — serves lints, formatting, code actions and the outline. The **types** are not in it. `larvae-lsp` — the binary — is the same server with Luau's analysis frontend plugged into it, which is what adds hover, completion and type diagnostics. And `larvae analyze` is nothing but a launcher for that binary: without it the command has no analyzer to run.
>
> So an install with only the first crate leaves you with a linter that works and an editor that silently gives you no types, with nothing on screen saying why.

## Through a tool manager

**If you installed through a tool manager**

```bash
larvae self install          # copies to ~/.larvae/bin and fetches the two missing files
larvae self update           # later, for the newest release
larvae self code             # completion and hover on larvae.toml itself
larvae self sync-luaurc      # write [aliases] into .luaurc, as paths an editor can follow
larvae self sync-luaurc --check   # write nothing; exit non-zero if it would change
```

rokit, aftman and ember install the one binary their manifest names, so `larvae-lsp` and its analyzer library never land beside it. `larvae self install` fetches them — the three files ship in one release archive. If it cannot, it says so rather than leaving the editor half-served.

> **larvae-lsp 0.8.0 is not on crates.io**
>
> Only if you are pinning an old version: 0.8.0 of the *server* crate was never published, because its build script rewrote a vendored source in place and cargo refuses to package a build that modifies its own sources. Install **0.8.1**, or take 0.8.0 from the git tag.

Whichever route you took, the test is the same: open a Luau file, hover a local, and look for a type. No type means the second binary is missing, whatever the install said. Kalo's own editors pass that test because the server behind them is `larvae-lsp`.

## Taking only the part you want

You do not have to adopt all of it. Three switches let larvae own one job and stay out of the way on the others — which is worth knowing before you run anything, because "try the require rewriter, keep stylua" is a perfectly good first step.

**Three ways to say "not this part"**

```toml
[fmt]
enabled = false     # keep stylua; larvae never rewrites a file

[lint]
enabled = false     # keep selene

[lsp]
claim_only = true   # the editor server answers only for worm-claimed files,
                    # so stock luau-lsp keeps every plain .luau document
```

`enabled = false` says it in the config rather than by not running the command — which matters, because the editor reads the same config and would otherwise keep formatting on save. `claim_only` is how larvae runs *beside* luau-lsp rather than instead of it.

**Key points**

- `cargo install larvae` and `cargo install larvae-lsp`: the second carries the types.
- Installed through rokit, aftman or ember? `larvae self install` fetches the two files the manifest did not name.
- `larvae self code` gives your editor completion and hover on `larvae.toml` itself.
- `[fmt] enabled = false`, `[lint] enabled = false` and `[lsp] claim_only = true` let larvae take one job and leave the rest.
