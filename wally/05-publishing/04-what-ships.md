# What actually goes in the archive

*include is not a whitelist, whatever the manifest says, so check before you publish.*

Wally · Publishing and Registries · 10 min

---

`wally publish` uploads a zip of your project directory, and two manifest keys decide what goes into it: `include` and `exclude`. They look like a symmetric pair — one names what to keep, the other names what to drop — and they are not. Getting this wrong is how a 12-file library ships someone's entire `assets/` folder.

**The project on disk**

```text
drop-table/
  wally.toml
  README.md
  LICENSE
  src/
    init.luau
    weights.luau
  tests/
    drop-table.spec.luau
  art/
    icon.png
    banner.png
```

Ten files. A consumer needs five of them.

**wally.toml**

```toml
[package]
name = "biff/drop-table"
version = "1.2.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"
include = ["src/**", "wally.toml"]
```

Read as a whitelist, this manifest ships three files. Read as Wally actually implements it, it ships all ten.

> **`include` is not a whitelist**
>
> The manifest's own doc comment says only files matching `include` are included. The filter does something else: it keeps a file if `include` matches it, **and otherwise keeps it unless `exclude` matches it**. So `include` can only ever add, and a short `include` list removes nothing at all. `exclude` is the half that can actually make an archive smaller.

Say it as a rule, because it is short: **a file ships unless `exclude` matches it.** `include` exists to rescue a file that an `exclude` glob would otherwise have caught — that is the job it is genuinely good at, and it is the only one. If you want a smaller archive, every line you write goes in `exclude`.

## See it before you send it

**wally package**

```bash
$ wally package --list
$ wally package --output drop-table.zip
```

`--list` builds exactly the archive `publish` would send and prints its contents instead of uploading. `--output` writes the zip so you can open it.

This is the only honest answer to "what is in my package?", and it costs nothing: no login, no network, no version bump. Run it before a first publish and read the list. Immutability means the archive you send is the archive that exists forever, so a minute here is cheap.

How wrong does it get? A real, popular package on the public index lists **six `include` entries** and ships **270 files and 1.15 MB** — over half the 2 MiB cap, in a library whose source is a fraction of that. Nobody chose that; the manifest reads as though it packs six things. Whoever wrote it believed the doc comment, which is exactly the trap this lesson exists for.

**Key points**

- A file ships unless `exclude` matches it. `include` only ever adds files back.
- A short `include` list shrinks nothing, however much the manifest's doc comment implies otherwise.
- Real packages ship hundreds of files by accident — 270 files and 1.15 MB against a 2 MiB cap is a live example.
- `wally package --list` builds the real archive and prints it, with no login and no network.
- Check the list before a first publish: the archive you send is immutable.
