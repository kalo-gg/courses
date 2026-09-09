# Installing a worm

*Where a worm comes from — a release, a crate, or a directory on disk — and the two commands that fetch it.*

Larvae · Worms & Extensions · 10 min

---

A worm arrives in `larvae.toml` as a table under `[worms]`, keyed by a name larvae will use for everything the worm adds. The table says where the worm comes from — a GitHub release, a crate, or a directory on disk — and, for a release, which version. Naming it is not fetching it; that is a second step, and on purpose.

**Naming a worm**

```toml
[worms]
luaux = { repo = "larvae-luau/luaux-worm", version = "^" }
```

`version` is required beside `repo`: `"^"` takes the newest release on every install, a number such as `"0.1.0"` holds one, and a range such as `"^0.1.0"` does what you would expect.

> **A worm is a table, not a string**
>
> `luaux = "larvae-luau/luaux-worm@0.1.0"` was the spelling before v0.5.0 and larvae now refuses it outright — helpfully, with the table form written out for you. You will still see the string form in older configs and in some READMEs, including larvae's own example file.

> **Naming a worm does not fetch it**
>
> `larvae worm install` (or `larvae worm i`) downloads every worm the config names; `larvae worm add luaux` writes the entry for you at its newest release. Naming one in `larvae.toml` on its own only declares the intent.

## Three sources, one key

**From a release**

```toml
[worms.markup]
repo = "someone/markup-worm"
version = "2.1.0"
asset = "markup-worm.zip"   # which file to take from the release
```

> **The `asset` default has three names, and the platform one is first**
>
> Leave `asset` out and larvae looks for `<name>-worm-<arch>-<os>.zip`, then `<name>-worm.zip`, then `worm.zip`, in that order. The platform name comes first because a **native** worm ships one artifact per platform; a portable wasm or Luau worm has no such asset, so the plain names answer for it.
>
> This matters if you are the one publishing. Name the release asset wrong and every install fails.

**From crates.io, or from disk**

```toml
[worms.luaux]          # a crate: built from source on the installing machine
cargo = "luaux-worm"
version = "0.1.0"

[worms.mine]           # a worm being developed: nothing to fetch
path = "build/myworm"
```

`cargo install` builds from source on the machine that runs it, so one published crate serves every platform and the author uploads no per-platform zip at all. The binary carries its own `worm.toml` and hands it back over the pipe, because `cargo install` ships no data files. `larvae worm add --cargo` writes this form.

The key is the repository name with `-worm` removed, so `luaux-worm` becomes `luaux` — which is what `larvae worm add luaux` writes, because `luaux` is a short name larvae knows and expands to `larvae-luau/luaux-worm` with `version = "^"`. That key is then the namespace for everything the worm adds, which the next lesson is about.

**Key points**

- A worm is a table under `[worms]`: `repo` with a `version`, or `cargo`, or `path`.
- `larvae worm add` writes the table; only `larvae worm install` downloads, and the editor never does.
- `asset` defaults to the platform-named zip first, because a native worm ships one per platform.
- The key is the repo name minus `-worm`, and it is the worm's namespace from then on.
