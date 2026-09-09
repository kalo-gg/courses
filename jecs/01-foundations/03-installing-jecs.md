# Installing JECS

*Wally, the release rbxm and roblox-ts: three ways to get jecs 0.11.0 into a project.*

JECS · ECS Foundations · 8 min

---

There are three supported ways to get JECS into a project. Pick the one that matches your toolchain: Wally if you use Rojo, the `.rbxm` if you work only in Studio, npm if you write roblox-ts. Whichever you choose, the result is one ModuleScript named `jecs` that both server and client can require, which is why it lives under `ReplicatedStorage`.

## 1. Wally (Rojo projects)

**wally.toml**

```toml
[dependencies]
jecs = "ukendio/jecs@0.11.0"
```

Then run `wally install`. Wally puts the package in `Packages/`, which your Rojo project usually maps to `ReplicatedStorage.Packages`.

Wally writes every dependency into `Packages/`, and your Rojo project file maps that folder into `ReplicatedStorage.Packages`. Pin an exact version: `0.11.0` is the release this course tracks, and JECS has changed its API between minor versions more than once.

**Requiring it in Roblox**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local jecs = require(ReplicatedStorage.Packages.jecs)
```

## 2. rbxm (Studio only)

Recent GitHub releases ship a `jecs.rbxm` asset (0.11.0 does; the odd older one such as 0.10.2 has no assets at all). Download it from the [releases page](https://github.com/Ukendio/jecs/releases), drag it into Studio and place the resulting `jecs` ModuleScript in `ReplicatedStorage` (so both server and client can require it).

## 3. roblox-ts

```bash
npm install @rbxts/jecs
```

**TypeScript usage**

```typescript
import { world, pair, ChildOf } from "@rbxts/jecs";

const w = world();
const Position = w.component<Vector3>();
const e = w.entity();
w.set(e, Position, new Vector3(0, 5, 0));
```

The TypeScript typings live in the same repository (`src/jecs.d.ts`) and follow the same version numbers as the Luau release.

> **What about pesde?**
>
> As of September 2026 there is **no official JECS package on the pesde registry**. Community mirrors exist (for example `marked/jecs`), but they are not published by the JECS author. If you use pesde, either depend on the Wally package through pesde's Wally support or vendor the single `jecs.luau` file yourself.

**Key points**

- Install with Wally (`ukendio/jecs@0.11.0`), the release `jecs.rbxm`, or `@rbxts/jecs` for roblox-ts.
- Wally installs into `Packages/`, which Rojo maps to `ReplicatedStorage.Packages`.
- Pin exact versions; JECS's API has moved between minor releases.
- There is no official pesde package; use Wally through pesde or vendor `jecs.luau`.
