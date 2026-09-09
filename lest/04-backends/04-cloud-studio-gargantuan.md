# The three engines

*One suite, two Roblox contexts that are not the same, and the experimental third engine.*

Lest · Backends · 12 min

---

`cloud`, `studio` and `gargantuan` all run your specs in a real engine, and they share a bundler: the same `[place] rojo` delegation, the same CLI-side snapshots. What differs is *which* engine, and what that engine thinks it is.

## Reaching studio at all

Studio is not something you configure a suite into — the suite stays `backend = "cloud"`, because that is what CI must use, and `--backend <BACKEND>` moves it for one run. There are no clicks and no plugin: Lest bundles the suite, launches Studio on the place — the `[place] file` when you have one, otherwise the published place — through its command-line interface, waits, and decodes the results from Studio's output file when Studio quits itself. Every run pays a boot of roughly 15–45 seconds, which is why watch mode never includes a studio suite.

**…run locally**

```bash
lest run engine --backend studio    # the same suite, in a launched Studio
lest run unit --backend lune       # or: does this suite behave in another runtime?
```

`--backend` forces every selected suite onto one backend for a single run. Precedence is **CLI flag > suite setting > top-level default** — it is a debugging override, not a substitute for declaring the right backend.

## Where cloud and studio disagree

| Asked from a spec | `cloud` | `studio` |
| --- | --- | --- |
| Runs as | A server script on a real game server | An edit-mode script, like the command bar |
| Permission level | GameScript | The command bar's — plugin-level APIs work |
| `RunService:IsStudio()` | `false` | `true` |
| `RunService:IsServer()` | `true` | `true` |
| `RunService:IsClient()` | `false` | **`true`** — an edit-mode quirk |
| `RunService:IsEdit()` | *throws* — plugin security | `true` |

An edit session acts as both contexts at once, which is why `IsClient()` and `IsServer()` are *both* true there.

> **Branch on `IsStudio()`, never on `IsServer()`, `IsClient()` or `IsEdit()`**
>
> This is the rule that makes one engine suite safe to run on studio locally and cloud in CI. `IsServer()` is true in both, so it tells you nothing. `IsClient()` differs — but for a reason that has nothing to do with which backend you are on, so a spec that branches on it behaves one way at your desk and another way on the pull request. And `IsEdit()` is a plugin-security method: the same call that returns `true` under studio *throws* at cloud's server permission level.

- **DataStores** follow game-server rules on cloud and edit-mode rules under studio, on a *published* place.
- **Plugin-level APIs differ the other way**: the command bar's permission is higher than a game server's, so an API needing plugin security works under studio and errors under cloud.
- Instances and services behave the same in both, which is why one suite serves both.

## gargantuan

[Gargantuan](https://github.com/teamfireworks/gargantuan) is an independent, Roblox-shaped engine scripted with Luau — a `game` DataModel, Instances, Signals, a `task` library. Lest bundles the suite with the same bundler, spawns `gargantuan --script <bundle> --headless`, and decodes sentinel-framed events from its stdout.

**Pointing at an engine you built**

```toml
[suites.engine-gg]
include = ["tests/gargantuan/**/*.spec.luau"]
backend = "gargantuan"
default = false

[gargantuan]
binary = "vendor/gargantuan/build/gargantuan"
```

With no `binary` configured, Lest looks for `gargantuan` on your `PATH`.

> **Experimental, stated plainly**
>
> The engine is pre-release and has **no tagged releases**, so you build it from source — that is what `[gargantuan] binary` is for. Its API surface is still filling in, so specs will find `not yet implemented` edges: **those are engine facts, not test failures**. It is excluded from watch mode and from `$CI` auto-enable, so it only ever runs when you name its suite.

**Key points**

- An engine suite stays `backend = "cloud"`; `lest run engine --backend studio` moves it for one run.
- Cloud is a server script on a game server; studio is an edit-mode script at the command bar's permission level.
- Branch on `IsStudio()` only: `IsServer()` is true in both, `IsClient()` flips for the wrong reason, `IsEdit()` throws on cloud.
- Gargantuan is a headless third-party engine you build yourself; `not yet implemented` is an engine fact.
