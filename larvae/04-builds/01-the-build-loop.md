# The build loop

*What larvae process reads and writes, the generator that keeps line numbers, and the Rojo project it derives.*

Larvae · Builds & Profiles · 10 min

---

`larvae process` is the command the rest of this module hangs off. It reads an input directory, rewrites every require into the target shape, applies whatever rules the config names, and writes a mirror of the tree to an output directory — plus one more file, the derived Rojo project you serve. `[process]` is the table that says what to read, where to write, and how the output should print.

**[process] in full**

```toml
[process]
input = "src"                  # a directory, or a list of them
output = "dist"                # a mirror of input with the transforms applied
include = ["**/*.luau"]        # which found files are transformed; the rest are copied
exclude = ["**/*.spec.luau"]   # skipped fully: not transformed, not copied
generator = "retain-lines"     # or "dense", or "readable"
quotes = "preserve"            # quote style for the require strings it writes
cache = true                   # incremental builds keyed by content hash
cache_dir = ".larvae"          # also where the derived Rojo project lands
strip_flags = true             # drop -- larvae: allow(...) comments from the output
```

## Three generators

| `generator` | Output | When |
| --- | --- | --- |
| `retain-lines` | Every line number kept | The usual choice: a stack trace in Studio points at the source |
| `dense` | Minified, tuned by `[minify]` | Release builds — the Minify and bundle lesson |
| `readable` | Formatted | Output a human is going to read |

`retain-lines` is the default, so "the usual choice" is also what you get for writing nothing.

Keeping line numbers is the choice that makes a build debuggable: with `retain-lines`, line 40 of a stack trace is line 40 of the file you wrote, however much the rewriter changed on it. The cache is what makes it fast enough to run on every save — `larvae process --watch` rebuilds only what changed, mirrors deletions, and keeps the last good output for a file that does not lex, so a half-typed save does not spread require failures into a live Studio session.

**The file it writes for rojo**

```toml
[rojo]
project = "default.project.json"              # read: where mounts come from
build_project = ".larvae/build.project.json"  # written: the same tree, pointing at the output
```

Both are found by name when unset. `larvae init` offers to gitignore `.larvae` because this file and the cache live there.

**The loop**

```bash
larvae process --watch                  # rebuild on every save, mirror deletions
rojo serve .larvae/build.project.json   # serve the derived project; larvae never runs rojo
```

> **The derived project is not a copy**
>
> It is your project file's tree with every `$path` remapped from the input directory to the output one and everything else copied through, regenerated on every build — so it never drifts from the one you edit, and it is never committed. This is the one project file instead of two that the README argues for against darklua.

Two lists, two questions. `include` asks which of the found files get transformed; `exclude` asks which are not found at all. A data file that matches neither is copied — a build is a mirror, not a filter — and a file that matches `exclude` is nowhere in the output.

**Key points**

- `[process]`: `input` and `output`, `include` for what is transformed, `exclude` for what is skipped, `generator` for how it prints.
- `retain-lines` keeps every line number so a stack trace points at the source; `dense` minifies; `readable` formats.
- `cache = true` keys incremental builds by content hash in `cache_dir`, which is also where `.larvae/build.project.json` lands.
- `larvae process --watch` rebuilds on save and keeps the last good output for a file that does not lex; `rojo serve` the derived project.
