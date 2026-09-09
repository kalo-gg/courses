# Installing and building

*Install the binary, scaffold a config, compile src into build, and point Rojo at the output.*

LuauX · LuauX Foundations · 9 min

---

LuauX is a single binary. Install it with whichever toolchain manager the project already uses — there is no Roblox-side install and nothing to add to your place file.

**Installing**

```bash
lpm tool add luaux                    # LPM
rokit add luau-xml/luaux && rokit install
mise use github:luau-xml/luaux@0.2.0
nix profile add github:luau-xml/luaux
```

Or build from source with `cargo build --release`. Release binaries are published per platform.

**The pipeline**

```bash
# scaffold a config for the library you already use
luaux init --library vide

# compile src/**/*.luaux into build/**/*.luau
luaux build src build

# and while developing
luaux watch
```

Then point Rojo at `build/` rather than `src/`, so the game syncs the compiled output.

## luaux.toml

Settings live in `luaux.toml`. The `[build]` section says what is compiled and where it lands.

Leaving `out` unset writes each `X.luaux` beside itself as `X.luau`, which suits a project migrating one file at a time. Setting it collects every output under one directory instead, which is the shape Rojo wants to point at, and it is the one this course uses from here on. Either way the compiler only ever writes where the config told it to, and it never writes into `src/` unless you asked for that.

**luaux.toml**

```toml
[build]
in = "src"
out = "build"
include = ["*"]
exclude = ["**/*.spec.luaux"]
clean = true

[factory]
backend = "table"
create = "create"
```

`[factory]` says which library to emit calls into. The Backends module takes it apart; for now, this pair is Vide.

> **Compiled output is not source**
>
> `clean = true` deletes outputs whose source is gone, so a renamed or removed `.luaux` file does not leave a stale `.luau` behind. It is a good hint about how to treat the directory: never edit the `.luau` files, and keep the output out of version control. The `.luaux` files are the source.

**Key points**

- Install with LPM, Rokit, mise or Nix; there is nothing to add to the place.
- `luaux init --library <name>` scaffolds `luaux.toml`, and `<name>` is one of `vide`, `fusion`, `fluid` or `react`; `build` compiles, `watch` keeps compiling.
- `[build] in`/`out` say what is compiled and where; no `out` writes each file beside itself.
- Rojo points at the output directory; never edit or commit the `.luau` files.
