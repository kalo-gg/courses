# Installing Fusion 0.3

*Get the release into ReplicatedStorage, then learn to tell 0.3 apart from the tutorials that are not.*

Fusion · Fusion Foundations · 9 min

---

The documentation describes two: download `Fusion.rbxm` from the Releases page, or take the source zip and copy `src` into your project as `Fusion` — the second is the one a Rojo project wants, with require examples for Rojo, darklua and plain Luau. **Neither is Wally**, and Wally is never mentioned; but the repository ships a `wally.toml`, so Fusion is a published Wally package all the same.

**Or with Wally**

```toml
# wally.toml
[dependencies]
Fusion = "elttob/fusion@0.3.0"
```

Published to the UpliftGames registry as a shared-realm package. The path you require then depends on where your Wally packages land — usually `ReplicatedStorage.Packages.Fusion`.

> **Two names for one release**
>
> The git tag is `v0.3-beta` but the Wally package is `0.3.0`. They are the same code; Wally requires a plain semantic version, which `v0.3-beta` is not. Do not read the mismatch as two different builds.

## Check which version you actually have

**Fusion tells you itself**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)

print(Fusion.version) --> {major = 0, minor = 3, isRelease = true}
```

On the released `v0.3-beta` tag this reports minor 3 with `isRelease = true`. The sandbox loads that tag, so the next exercise prints the same.

> **The main branch is not 0.3**
>
> Fusion's `main` branch reports `{major = 0, minor = 4, isRelease = false}` and adds APIs — `Tag` and `insert` among them — that no release has. If you copy `main` into your game, examples from the 0.3 docs will still work, but you will also find yourself reading about functions that will not exist for other people using the release. This course teaches the release.

There is a second, larger trap. Fusion 0.3 was released on **30 August 2024**, and the nineteen months of tutorials, videos and forum answers written before it describe Fusion 0.2, whose API is different in a way that is easy to miss: 0.2 calls have no scope argument.

If a snippet says `Fusion.Value(0)` or `New "TextLabel" { … }`, it is 0.2 and it will error on 0.3.

| If a tutorial says | 0.3 wants | Why |
| --- | --- | --- |
| `Fusion.Value(0)` | `scope:Value(0)` | Every constructor takes the scope that will own the object. |
| `thing:get()` | `use(thing)` or `Fusion.peek(thing)` | Reads either record a dependency (`use`) or deliberately do not (`peek`). |
| `Computed(function() return x:get() end)` | `scope:Computed(function(use) return use(x) end)` | The callback is handed its reader. |
| `Tag`, `insert` | — not in any release | Those are on `main`, heading for 0.4. |

**Key points**

- Install from the Releases page (`Fusion.rbxm`) or with Wally (`elttob/fusion@0.3.0`); the tag `v0.3-beta` and the package `0.3.0` are the same code.
- `Fusion.version` tells you what you are running; 0.3 reports `{major = 0, minor = 3, isRelease = true}`.
- A snippet with no scope argument, or a `:get()` call, is 0.2 and will not run on 0.3.
- `Tag` and `insert` are on `main` only; no release has them.
