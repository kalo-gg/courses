# Installing Vide

*Add Vide 0.4.1 with Wally, pesde or the release rbxm, require it, and read its version.*

Vide · Vide Foundations · 8 min

---

Vide is a single ModuleScript with its sources as children. There are three ways to get it into a project. This course is verified against **Vide 0.4.1**, released July 11, 2026.

| Route | What you do | Pick it when |
| --- | --- | --- |
| Wally | `vide = "centau/vide@0.4"` in wally.toml, then `wally install` | A Rojo project that needs 0.4.1 |
| pesde | `pesde add centau/vide` | A pesde toolchain and 0.4.0 is enough |
| build.rbxm | Drag the release asset into Studio | A Studio-only place with no package manager |

## Wally and pesde

**wally.toml**

```toml
[package]
name = "you/my-game"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
vide = "centau/vide@0.4"
```

Then run `wally install` and make sure your Rojo project maps `Packages/` into `ReplicatedStorage.Packages`.

```bash
rokit add pesde-pkg/pesde
pesde init
pesde add centau/vide
```

Vide publishes to the [pesde](https://pesde.dev/) index as well, and its own `rokit.toml` lists pesde above Wally. One catch as of this course: the newest Vide on pesde is **0.4.0**, so if you need 0.4.1 use Wally or the rbxm.

Whichever route you take, the result is one ModuleScript named `vide` with the library's sources as children, and it has to live where a client can require it, because UI code runs in LocalScripts. `ReplicatedStorage.Packages.vide` is where Wally and pesde put it and where every lesson's first line looks for it.

**The rbxm.** Releases since 0.4.0 attach a file called **`build.rbxm`** to the [GitHub releases page](https://github.com/centau/vide/releases) — not `vide.rbxm`, and earlier releases have no assets at all. Drag it into Studio and place the ModuleScript (already named `vide`) under `ReplicatedStorage`. UI runs on the client, so it must live somewhere clients can reach.

> **Vide is pre-1.0**
>
> Every Vide release is tagged as a prerelease on GitHub, so there is no "latest release" link to point a script at. Each minor version so far has carried breaking changes — the reference tab's **What changed** list is there for exactly this reason. Pin an exact version and read the changelog before bumping.

## Requiring it

```luau
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)

-- Pull out what you use; every lesson does this
local create = vide.create
local source = vide.source
local effect = vide.effect

print(vide.version.major, vide.version.minor, vide.version.patch) -- 0 4 1
```

`vide.version` is a table, handy for asserting the version your code expects.

> **Strict mode is on by default in Studio**
>
> In Studio, Vide runs extra checks called **strict mode**: effects run twice per update so mistakes show up early. Published places skip the checks. The Kalo sandbox keeps strict mode **off** so printed output is easy to follow; Strict mode and debugging, in Shipping a Vide App, turns it on deliberately and explains what it catches.

**Key points**

- Install with Wally (`centau/vide@0.4`), pesde (`pesde add centau/vide`, 0.4.0) or the `build.rbxm` attached to releases since 0.4.0.
- Put the module where clients can require it, usually `ReplicatedStorage.Packages.vide`.
- `vide.version` tells you what loaded; this course targets 0.4.1.
- Strict mode is on in Studio and off in production; the sandbox starts with it off.
