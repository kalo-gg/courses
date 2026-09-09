# Migrating from darklua, stylua and selene

*darklua, stylua and selene map onto larvae name by name; the three rules that moved and the two files that become one.*

Larvae · Larvae for a Team · 10 min

---

A darklua pipeline is a `.darklua.json` with a rules list, a generator, and two Rojo project files — one pointing at the source for sourcemaps and one at the build for serving. larvae accepts every darklua rule name, so the list pastes into `[rules]`; the generator has the same three names in larvae's spelling; and the second project file is the one larvae derives, so it goes.

**.darklua.json, before**

```json
{
  "generator": "retain_lines",
  "rules": [
    "compute_expression",
    "remove_unused_if_branch",
    { "rule": "inject_global_value", "identifier": "DEBUG", "value": false },
    { "rule": "convert_require", "current": { "name": "path" }, "target": { "name": "roblox" } }
  ]
}
```

**larvae.toml, after**

```toml
[requires]                    # convert_require lives here now
target = "roblox-string"

[defines]                     # inject_global_value lives here now
DEBUG = false

[process]
generator = "retain-lines"    # darklua's retain_lines

[rules]
compute_expression = true
remove_unused_if_branch = true
```

## The three that moved, and the two files

| darklua | larvae | Note |
| --- | --- | --- |
| `convert_require` | `[requires] target` and `[aliases]` | Plus the realm check darklua never had |
| `inject_global_value` | `[defines]` | One line per constant |
| `remove_spaces` | `[process] generator = "dense"` | Tuned by `[minify]` |
| `"generator": "retain_lines"` | `generator = "retain-lines"` | Same three names, hyphenated |
| the second project file | `[rojo] build_project`, derived on every build | Delete it; serve `.larvae/build.project.json` |
| `darklua process --watch` | `larvae process --watch` | Keeps the last good output when a file does not lex |

What the migration gains is the check darklua never had: `larvae check` walks the require graph with realms in mind, so the first run on a real repo usually finds a client file reaching into ServerStorage that has compiled fine for years — and would have failed in front of a player.

stylua and selene are the easy half, because larvae reads both files as they are. Leave them in place on day one, run `larvae fmt --check` to see what the three opinions would do, add `[fmt] recommended = false`, then move the settings into `[fmt]` and `[lint]` at your own pace — spelling selene's `error` as `deny` — and delete the two files last. `larvae init` writes the comments that say which file it is reading, so the order of the move is visible in the config.

> **Migrate in this order**
>
> 1. `larvae init` — it writes comments pointing at `stylua.toml` and `selene.toml`, not tables.
> 2. Paste `.darklua.json`'s rules into `[rules]`, moving the three that moved.
> 3. Delete the second project file and `rojo serve .larvae/build.project.json`.
> 4. `larvae check` — read what it finds before anything else changes.
> 5. Move `[fmt]` and `[lint]` in, one file at a time, and delete the old files last.

**Key points**

- Every darklua rule name is accepted; `convert_require`, `inject_global_value` and `remove_spaces` moved and say where.
- `retain_lines` becomes `retain-lines`; the second project file becomes the derived one larvae writes.
- stylua and selene files stay readable; `recommended = false` first, then move settings at your pace, `error` spelled `deny`.
- The first `larvae check` on a migrated repo is the realm audit darklua never ran.
