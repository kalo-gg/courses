# The editor server

*The lsp table: what the server answers, which files it claims, and where the types come from.*

Larvae · Larvae for a Team · 10 min

---

Everything so far was the CLI. The same config drives the editor: `larvae lsp` serves lints, formatting, code actions and the outline over stdio, and `larvae-lsp` — the second binary — plugs Luau's analysis frontend into the same server for hover, completion and type diagnostics. `[lsp]` is the table that says how much of that to answer, and for which files.

**What the server answers**

```toml
[lsp]
enabled = true         # false answers every request with nothing: another server owns the files
claim_only = false     # true answers only for worm-claimed files; stock luau-lsp keeps .luau
analyzer = true        # false keeps lints, format, actions and outline; hover, completion and types go
roblox_security_level = "roblox-script-security"   # or "none", "local-user-security", "plugin-security"
```

## Where the types come from

**The world the analyzer types against**

```toml
[lsp]
sourcemap = "sourcemap.json"                  # the rojo sourcemap it reads
sourcemap_autogenerate = true                 # the server runs the sourcemap command itself
rojo_project_file = "default.project.json"    # the project that command reads
definitions = ["types/globals.d.luau"]        # extra type definitions, after the platform globals
documentation = ["types/documentation.json"]  # luau-lsp documentation JSON, for hover
```

**The same world, in the terminal**

```bash
larvae analyze src --definitions types/globals.d.luau   # spawns larvae-lsp; nothing without it
larvae self code                                        # completion and hover on larvae.toml itself
```

The analyzer needs two things it cannot infer from the source: the shape of the DataModel, which the Rojo sourcemap gives it, and any globals your project defines, which `definitions` files give it. The server can run rojo's sourcemap watch itself, or stay out of the way of one you already run — `sourcemap_autogenerate = false`. `larvae analyze` takes the same definitions on the command line, so CI and the editor type against the same world.

| Table | Keys | Does |
| --- | --- | --- |
| `[lsp.completion.imports]` | `enabled`, `require_style` (`auto`, `always_relative`, `always_absolute`, `nearest_absolute`), `use_const` | Auto-imports a service or module the file has not required yet |
| `[lsp.inlay_hints]` | `variable_types`, `parameter_types`, `function_return_types`, `parameter_names` (`none`, `literals`, `all`) | Every one off by default |
| `[lsp.studio]` | `enabled`, `port` (3773) | The Studio plugin link; off, because it opens a socket |
| `[lsp.fflags]` | `enable_new_solver`, `enable_by_default`, `over` | Luau's own analysis flags |

> **Kalo runs this**
>
> Every editor in this course talks to `larvae-lsp` on Kalo's server, with a `larvae.toml` beside the documents that loads the LuauX worm — which is why a `.luaux` file here gets hover and completion, and why a document has to end in `.luaux` to be lowered at all. Initialized with no workspace root the server never finds that config, and every hover in markup answers `*error-type*`.

`larvae self code` is the last editor job, and it is about `larvae.toml` itself: it wires the config's JSON schema into the editor — through the TOML extension's settings when one is installed, or a `#:schema` line at the top of the file when not — so every key and every lint name, worm lints included, completes as you type it.

**Key points**

- `larvae lsp` carries lints, format, code actions and the outline; `larvae-lsp` adds the types.
- `enabled`, `claim_only` and `analyzer` are the three ways to answer less; `claim_only = true` is how larvae runs beside luau-lsp.
- The analyzer types against the Rojo sourcemap and your `definitions`; `larvae analyze` takes the same files.
- `larvae self code` makes `larvae.toml` itself complete as you type.
