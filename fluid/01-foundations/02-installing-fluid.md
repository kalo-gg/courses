# Installing Fluid

*One Wally line, an exact pin, and an honest look at what a release candidate means.*

Fluid · Fluid Foundations · 9 min

---

Fluid publishes to Wally, in the UpliftGames registry, in the `shared` realm. There is one line to add to `wally.toml`, one character in it that matters more than the rest, and one fact about which version you are getting.

**wally.toml**

```toml
[dependencies]
fluid = "ffrostfall/fluid@=0.7.0-rc.7"
```

UpliftGames registry, shared realm. `wally install` puts it under `ReplicatedStorage.Packages.fluid`. Note the `=`.

> **Without the `=`, that line is a range**
>
> Wally reads dependency requirements Cargo-style, so a bare `ffrostfall/fluid@0.7.0-rc.7` means `^0.7.0-rc.7` — "this or anything compatible above it". The moment `0.7.0-rc.8` or `0.7.0` appears in the index, `wally install` picks it up and your build changes under you. On a release candidate that is exactly the wrong behavior. `@=0.7.0-rc.7` is the exact pin.

## Which version am I actually teaching?

|  | Latest **tagged** release | What this course teaches |
| --- | --- | --- |
| Version | v0.6.5 | 0.7.0-rc.7 |
| Dated | 9 July 2026 | commit `a93f33a`, 25 July 2026 |
| Where the number comes from | a git tag and a GitHub release | `wally.toml` — the 0.7 line has no tag |
| Why pick it | it is stable | it is what the README and current examples describe, and 0.7 removed dev groups outright |

If you install the stable release, parts of this course will not match. Know which one you have.

Practically, this means two habits. Pin the exact version with `=` rather than accepting a range, and read the changelog before upgrading — it is maintained and follows Keep a Changelog, so it is the fastest first look.

**Requiring it**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

print(type(fluid.source)) --> function
```

In a Rojo project the `Packages` folder Wally writes is mapped into `ReplicatedStorage`, so every script reaches Fluid at the same address. Get the manifest right once and the require line is the same in every file you write afterwards.

**Key points**

- Fluid installs from Wally: `ffrostfall/fluid`, UpliftGames registry, `shared` realm.
- `@0.7.0-rc.7` is a **range**; `@=0.7.0-rc.7` is a pin. On a release candidate, pin.
- The latest tagged stable release is v0.6.5; this course teaches 0.7.0-rc.7, which has no tag.
- `wally install` writes `Packages/`, which Rojo maps into `ReplicatedStorage`.
- Read the changelog before you bump, and expect names to move before 1.0.
