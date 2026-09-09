# The CLI

*Five commands and where each belongs: the terminal, the watch loop beside Rojo, or CI.*

LuauX · Tooling & Adoption · 10 min

---

You installed LuauX in the Foundations module — LPM, Rokit, mise or Nix, a single binary, nothing in the place.

It has five commands, and the useful thing to know about each is where it belongs: in your terminal once, in the loop that runs while you edit, or in CI where it has to finish with an exit code. Two of the five are the bookends of a project — `init` at the start and `scan` during a migration — and the other three all compile the same files, differing only in what they do with the result.

| Command | What it does |
| --- | --- |
| `luaux build [src] [out]` | Compile `.luaux` to `.luau`. |
| `luaux check [src]` | Compile without writing — the CI form. |
| `luaux watch [src] [out]` | Rebuild on change, beside `rojo serve`. |
| `luaux init [dir] [--library]` | Scaffold a `luaux.toml`. |
| `luaux scan <path>...` | Scan `.luau` files for markup-shaped code — a migration aid. |

Paths default to `[build] in` and `out` from `luaux.toml`, so a bare `luaux build` works once the config exists. Every non-source file is **copied** into the output unless `include` or `exclude` say otherwise — which is what lets Rojo point at the build directory and still find your `.rbxmx` models and JSON beside the compiled Luau.

**The two you run outside the loop**

```bash
$ luaux init --library vide
wrote luaux.toml

$ luaux scan src
src/ui/Panel.luau:12   create "Frame" { … }
src/ui/Hud.luau:4      create "TextLabel" { … }
```

`init` writes the config once, at the start; `scan` reads the `.luau` files you have **not** converted and points at markup-shaped code. The report's exact wording is the CLI's — what matters is that it names a file and a line, and that it never looks at a `.luaux` file.

## The development loop

> **The development loop**
>
> `luaux watch` in one terminal, `rojo serve` in another, and Rojo's source directory set to the build output. You keep editing `src/`; Studio sees `build/`. Nothing else about your Rojo project changes.

**Two terminals**

```bash
# terminal 1: keeps build/ current as you edit src/
luaux watch

# terminal 2: syncs build/ into Studio
rojo serve
```

Paths come from `[build]`, so both commands are bare once `luaux.toml` exists.

**Key points**

- `build` compiles, `check` compiles without writing, `watch` keeps compiling.
- `init --library` scaffolds the config; `scan` reads `.luau` files for markup-shaped code.
- Bare commands read `[build] in` and `out`; non-source files are copied into the output.
- `luaux watch` in one terminal, `rojo serve` in the other, Rojo pointed at `build/`.
