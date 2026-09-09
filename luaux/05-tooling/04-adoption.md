# One file at a time

*Convert a project without a big bang: in place first, scan for candidates, a check in CI, a build directory last.*

LuauX · Tooling & Adoption · 11 min

---

Leaving `out` unset writes each `X.luaux` beside itself as `X.luau`, so a project can convert one file at a time with no change to its Rojo project at all. `luaux scan` reads your **`.luau` and `.lua`** files and reports where it finds markup-shaped code, which is how you decide what to convert next — point it at a directory of already-converted `.luaux` files and it reports nothing, because those are not the files it inspects.

**Beside the source, not in a build directory**

```toml
[build]
in = "src"
# no out: X.luaux -> X.luau, in place

[factory]
backend = "table"
create = "create"
```

## A migration that never breaks the build

The in-place phase is what makes the migration safe: the compiled twin sits exactly where the old file was, so Rojo's project file does not change and nothing else in the repo knows a conversion happened.

Two things to decide while you are there. The twins are generated, so either commit them for now or have CI run `luaux build` before `rojo build`. And when you finally move to a build directory, set `clean = true` in the same change, so a file you rename or delete later does not leave a stale copy in `build/`.

**.github/workflows/ci.yml — the gate**

```text
steps:
  - uses: actions/checkout@v4
  - run: rokit install          # installs luaux and rojo from rokit.toml
  - run: luaux check src        # compiles everything, writes nothing
  - run: rojo build -o place.rbxl
```

Install Rokit the way your CI already does; the LuauX step is one line and it fails the job on the first bad class name.

**What you can do now**

- Install LuauX and run `build`, `check`, `watch`, `init` and `scan`.
- Sit a compile step beside `rojo serve`, and a `luaux check` in CI, without changing the Rojo project until the end.
- Read a LuauX diagnostic and trust its line numbers.
- Get editor support from `luaux-worm`, and migrate a project one file at a time.
