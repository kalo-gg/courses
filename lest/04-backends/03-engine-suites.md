# Engine suites

*A real place, an API key that never touches your config, and a suite that stays out of the local loop.*

Lest · Backends · 12 min

---

`default = false` plus `$CI` is the pattern worth stealing. The suite runs only when named (`lest run engine`) or when `$CI` is set to anything but empty, `0` or `false` — so a slow cloud suite stays out of your local loop and still gates every pull request.

**An engine suite**

```toml
[suites.engine]
include = ["tests/engine/**/*.spec.luau"]
backend = "cloud"
default = false          # opt in locally; auto-enabled when $CI is set

[place]
universe_id = 1234567890
place_id = 9876543210
```

> **Two backends are exempt from the `$CI` half**
>
> `$CI` auto-enables `default = false` suites on every backend **except `studio` and `gargantuan`**. Studio is a GUI application that refuses to run in CI at all — name its suite there and you get **exit 2**, which by this course's own exit-code contract means *Lest broke*, not *tests failed*. Gargantuan needs an engine binary you built locally, so it is opt-in by name too. Declare an engine suite as `backend = "cloud"`, and reach studio with a flag; *The three engines* shows how.

## What you need

- **A published place to run against** — a small dedicated test place is the usual arrangement.
- **The universe and place ids**, from the Creator Dashboard URL. Not secret, so they belong in `lest.toml`.
- **An API key** with the universe-places Luau-execution scope — plus the write scope if you set `[place] file`.

> **The key is read from the environment only**
>
> Never from `lest.toml`, and never printed. `ROBLOX_API_KEY` or `LEST_API_KEY`, or a `.env` at the project root that is loaded automatically — and that you do not commit. The ids being in the config and the key being outside it is a deliberate split, not an oversight.

## Keeping the place current

**Pinning what runs**

```toml
[place]
universe_id = 1234567890
place_id = 9876543210
file = "test-place.rbxl"       # uploaded by cloud, launched by studio
rojo = "default.project.json"
```

With `file` set, every cloud run makes sure the place holds exactly that file: it is uploaded as a new saved version — skipped when its content hash matches the last upload — and every task is pinned to that version. Without it, tasks run against whatever the place currently holds, *which is fine for an empty place and a foot-gun for a populated one*: forget to publish after a fixture change and the suite quietly tests last week's place.

> **`rojo` makes a require reach the live module**
>
> With it set, a string require whose target the project file maps to a **ModuleScript** is delegated: instead of bundling a private copy, the generated require walks to the live instance and hands it to the engine's own `require`. The spec and the place's code then share one module through the engine's cache — so `require('../packages/thing/src')` reaches the same singleton the place's scripts see, with full static types in your editor.

Only targets that the project file maps to a **ModuleScript** delegate; a mapped `Script`, a folder or an unmapped file bundles as before, and `lest/core` never delegates. A missing instance at run time fails with the mapped path, which pairing `rojo` with `file` removes. The native, lune and lute backends ignore the key.

**Key points**

- An engine suite is `backend = "cloud"`, `default = false`: named locally, automatic under `$CI`.
- `studio` and `gargantuan` are exempt from `$CI`; reach them by name or by flag.
- Ids in `[place]`; the key in the environment or `.env`, never in the config.
- `[place] file` pins every task to one uploaded version; `rojo` lets a require reach the live ModuleScript.
