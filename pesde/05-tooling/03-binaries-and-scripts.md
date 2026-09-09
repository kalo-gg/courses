# Binaries, scripts and pesde run

*Packages you run rather than require, one-off execution with pesde x, and how pesde run resolves its argument.*

pesde · Workspaces, Tooling & CI · 10 min

---

A package exporting a `bin` is a **binary package** — a program rather than a library. Only the `luau` and `lune` environments can export one.

**Three ways to run one**

```bash
pesde x pesde/hello           # one-off, no project needed
pesde x -t lune acme/builder  # …when it is only published for Lune

pesde add pesde/hello         # or install it,
pesde install
hello                         # and it is on your PATH
```

One-off, or installed once and kept on your `PATH`.

An installed binary lands on your `PATH`, scoped to the nearest `pesde.toml`; workspace members use the root's. `pesde x` resolves the **`luau`** target into a temporary root, and refuses a Roblox one.

> **`_G.PESDE_ROOT`**
>
> A binary package can read `_G.PESDE_ROOT` for **its own** install root — the directory holding the `pesde.toml` of the package being run, or a temporary directory under `pesde x`. It is how a tool finds files it shipped alongside itself, not how it finds the project it is acting on; for that, use the working directory.

## `pesde run`

`pesde run` resolves its argument in four steps, and knowing the order saves confusion: no argument runs your own `target.bin`; otherwise it tries a **package name or dependency alias** first, then a name in `[scripts]`, and only then treats the argument as a **path**.

So a dependency aliased `build` wins over a `[scripts] build` entry. Anything after `--` is passed along.

**Scripts**

```toml
[scripts]
build = "scripts/build.luau"
test = "scripts/test.luau"
```

Run with `pesde run build`. Scripts are executed by **Lune**.

**Key points**

- A `bin` under `luau` or `lune` makes a binary package; `pesde x` runs one without a project, `-t lune` when it is Lune-only.
- Installed binaries land on your `PATH`, scoped to the nearest `pesde.toml`.
- `_G.PESDE_ROOT` is the package's own root, not the project's.
- `pesde run` tries `target.bin`, then a package or alias, then `[scripts]`, then a path — and `--` passes arguments through.
