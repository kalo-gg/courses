# The first hour

*What larvae init writes for your project, the six commands of the first hour, and the Rojo handoff.*

Larvae · Larvae Foundations · 10 min

---

You have larvae on your PATH and no config. The first hour is six commands, and the first of them writes the config for you — shaped by what the project already has, which is the reassuring part if you are arriving from stylua and selene. The order matters: two of the six change files, and both come after the ones that only look.

**Six commands, in order**

```bash
larvae init                            # writes larvae.toml, offers to update .gitignore
larvae fmt --check                     # writes nothing; expect a diff, see Formatting
larvae lint                            # reads the selene.toml you already have
larvae check                           # the realm violation you did not know you had
larvae process                         # writes dist/ and .larvae/build.project.json
rojo serve .larvae/build.project.json  # larvae never runs rojo itself
```

Run `--check` before `fmt` on a real repo. It exits non-zero if a file would change and touches nothing, so it is also the CI invocation.

## What `larvae init` writes

`larvae init` does not write one fixed file. It looks at what the project already has, and the answer is reassuring if you are arriving from stylua and selene:

| In your project | What `init` writes |
| --- | --- |
| `stylua.toml` | A comment: *formatting follows stylua.toml, which larvae reads as it is* |
| `selene.toml` | The same comment for `[lint]` |
| Neither | `[fmt] recommended = true` plus `column_width`, `indent_type`, `indent_width`, `quote_style`, and `[lint] recommended = true` with `std = "roblox"` |
| A `.luaurc` with no `lint` key | `"lint": { "*": false }` inserted into it, leaving every other byte alone |
| A `.gitignore` | An offer to add `.larvae` to it |

It writes `recommended = true` even though that is the default. The value is not the point; the key is — a key you can see is a key you can turn off.

**.luaurc after init**

```json
{
  "aliases": { "shared": "./src/shared" },
  "lint": { "*": false }
}
```

The one line `init` inserts, leaving every other byte alone — and only when the file exists and had no `lint` key.

> **The `.luaurc` handover is conditional**
>
> larvae covers all 28 of Luau's compiler lints — keeping Luau's level for each, and reporting the narrower, type-free case wherever Luau would consult the type checker and larvae cannot — so you do not want both reporting. But `init` only edits `.luaurc` when there is one **and** it does not already set `lint`. No `.luaurc` and it creates none: making a file just for this would be overreach. A `.luaurc` that already has an opinion and it prints *"already sets lint, leaving it alone"* and moves on.
>
> If you never had a `.luaurc`, nothing was handed over, and Luau's own lints are still on. That is fine — just do not assume it was handled.

## The Rojo handoff

> **The Rojo handoff is the point**
>
> The usual darklua setup keeps two near-identical project files: one pointing at the source for sourcemaps, one at the build output for serving. larvae **derives** the second from the first and keeps it current, so you edit one file. It writes `.larvae/build.project.json` and stops there — serving is rojo's job.
>
> `.larvae` is also the incremental cache directory (`[process] cache_dir`), which is why `init` offers to gitignore it.

Once that loop works, `larvae process --watch` is the daily driver: it rebuilds on every save and mirrors deletions, and when a file does not lex it keeps the last good output — so a half-typed save does not spread require failures into a live Studio session.

**Key points**

- `larvae init` writes comments pointing at a `stylua.toml` or `selene.toml` it finds, and tables when it finds none.
- It hands Luau's lints over in `.luaurc` only when the file exists and had no `lint` key.
- Look three times (`fmt --check`, `lint`, `check`) before `process` writes anything.
- `larvae process` writes `dist/` and `.larvae/build.project.json`; `rojo serve` that file — larvae never runs rojo.
- `larvae process --watch` rebuilds on save and keeps the last good output when a file does not lex.
