# Wally dependencies and [place]

*A Wally package needs the sourcemap generator to get an entry point; [place] and roblox_server round out the setup.*

pesde · Roblox Projects · 10 min

---

A Wally dependency installed through pesde leans on the **`sourcemap_generator`** script for more than editor comfort. A Wally package has no `pesde.toml`, so nothing inside it says which file is the library. pesde runs the script with the package's directory as the first argument through `process.args`, reads the sourcemap it prints, and takes the first `.lua` or `.luau` file it finds — *that* is how the dependency gets an entry point.

**Everything a Wally dependency needs**

```toml
[wally_indices]
default = "https://github.com/UpliftGames/wally-index"

[dependencies]
signal = { wally = "sleitnick/signal", version = "^2.0.0" }

[scripts]
sourcemap_generator = ".pesde/scripts/sourcemap_generator.luau"
```

The Rojo one is in `pesde-pkg/scripts`; `pesde init` writes this line for you when you pick a scripts package, and `pesde install` puts the file there.

> **The warning understates the damage**
>
> With no `sourcemap_generator`, pesde prints `no sourcemap_generator found in project. wally types will not be generated` — which sounds like a types problem. It is not only that. The linker falls back to a placeholder export path (`____pesde_no_export_file_found`) that does not exist, so the Wally dependency has no working entry point either. If a Wally package requires as `nil` or fails to resolve, this script is the first thing to check.

## `[place]`, and the one linker that reads it

**`[place]`**

```toml
[place]
shared = "game.ReplicatedStorage.Packages"
server = "game.ServerScriptService.Packages"
```

Two entries because the two Roblox targets are synced to different places.

`[place]` is consulted in exactly one situation: when a linker has to cross from one target's packages directory into another's — a `roblox_server` package that depends on a `roblox` one, or a `roblox` package that depends on a `roblox_server` one. Those two directories are mounted under different services, so the crossing cannot be written as a path relative to the linker, and pesde emits the datamodel path from `[place]` instead. A root project's own direct dependencies never read it, whatever their origin — registry, Wally or git — and 0.7.2's switch to string requires did not change that either way. Declare both entries anyway: they cost two lines, and without them a cross-target package resolves to nothing in a running game.

## About `roblox_server`

> **It is not an optimization**
>
> Using `roblox_server` to keep a server-only dependency off the client sounds sensible and is discouraged: it complicates linking, makes the package harder to use, and on a public registry is redundant, since anyone can download it anyway. The saving on client sync is a micro-optimization you will not measure.

It exists for **private registries** — internal packages such as configs or genuinely sensitive code that you do not want clients to see. If you are publishing publicly, use `roblox`.

**Key points**

- A Wally package has no `pesde.toml`; the `sourcemap_generator` is how it gets an entry point, not just types.
- Wally specifiers read `[wally_indices]`.
- `[place]` names where each target's packages sit in the datamodel, and only a linker crossing targets (`roblox_server` → `roblox`, or back) reads it — a project's own direct dependencies never do, whatever their origin.
- `roblox_server` is for private registries, not for hiding code from clients.
