# Lest reference

*Lest 0.6.0, released August 21, 2026. Checked on September 7, 2026.*

---

## Structure

Registration happens first; execution second.

### `A spec file`

```luau
--!strict
local Lest = require('@lest')
local describe, it, expect = Lest.describe, Lest.it, Lest.expect

-- tests

return nil
```

Specs are ordinary modules named `*.spec.luau`. Lest collects tests through registration, so the return value is never read.

### `describe(name, body)`

```luau
describe('parser', function () … end)
```

Groups tests. The body runs **immediately, at collection time** — its only job is registration.

> An error thrown in a body fails collection: it is re-raised as `error while collecting describe('name')`, which propagates out of loading the spec — the **whole file fails to load and nothing in it runs**, including blocks that collected cleanly. Calling `describe`, `it` or a hook after the run has started is a hard error.

### `it(name, fn)`

```luau
it('sums line items', function () … end)
```

Registers one test. Its body runs later, during the run.

### `The full name`

```luau
parser errors reports the line number
```

The describe path joined with the test's own name by **single spaces**. This is what `-t` filters against.

> Two tests may never share one within a file — a collection-time error, because they would collide on snapshot keys. The ` › ` glyph in the Slowest Tests block and JUnit `classname` is decoration, not part of the name.

### `Test doubles`

```luau
checkout.new({ clock = function () return 0 end, stock = fakeStock })
```

Lest ships no mock, spy or module-patching API: code takes its collaborators as arguments and a spec passes a fake — a working stand-in, or one that records what it was asked into a table you assert on with `toEqual`.

> Fake time, randomness, network and storage at a boundary you own; never the thing under test, and never the engine — engine behavior belongs in an engine suite.

## Lifecycle hooks

Each scoped to the block that registers it.

### `Lest.beforeAll(fn)`

```luau
beforeAll(function () … end)
```

Once, before the first test in the block. Multiple hooks run in registration order.

### `Lest.beforeEach(fn)`

```luau
beforeEach(function () … end)
```

Before every test in the block, **outermost first**.

### `Lest.afterEach(fn)`

```luau
afterEach(function () … end)
```

After every test in the block, **innermost first**.

### `Lest.afterAll(fn)`

```luau
afterAll(function () … end)
```

Once, after the last test in the block. Multiple hooks run in **reverse** registration order — teardown is LIFO.

### `A failing beforeAll`

```luau
—
```

Fails every test in its block rather than reporting a cascade of unrelated assertion failures.

> Descendant blocks of a failed `beforeAll` do not run their own `beforeAll` either. Same-named sibling blocks get independent hooks — blocks are identified by position, not by label. An entered block still runs its own `afterAll` hooks after a failed `beforeAll`, since partial setup has to be undone; a failing `afterAll` is reported as a failure named `(afterAll)` under the block.

## Modifiers

### `it.skip`

```luau
it.skip('name')
it.skip('name', fn, 'reason')
```

Reported, never executed. The body is optional; a third argument gives a reason reporters surface.

> The reason is also how a flaky test is quarantined honestly: the body stays and every report prints why it is not running.

### `it.todo`

```luau
it.todo('name')
```

A test you intend to write. Takes no body — passing one is an error. Reports as skipped with the reason `todo`.

### `it.only`

```luau
it.only('name', fn)
```

Focuses. When any test in a file is focused, only focused tests run and the rest report `skipped by .only`.

> Focus is **per spec file**. It composes with `-t`: the filter narrows first, then focus narrows what remains.

### `describe.skip / describe.only`

```luau
describe.skip('block', fn)
```

The same, for a whole block. `describe.skip` gives each test the reason `describe.skip`, and bodies are still collected so names and nesting still appear.

### `xit`

```luau
— removed in 0.6
```

`it.skip` replaces it with the same signature, so the migration is a rename.

## Matchers

Dot-calls, chainable, negated as toNot* — `not` is a Luau keyword.

### `toBe / toNotBe`

```luau
expect(1 + 1).toBe(2)
```

Identity — plain Luau `==`. Two tables match only if they are the same table.

### `toEqual / toNotEqual`

```luau
expect(parse('a=1')).toEqual({ a = 1 })
```

Deep structural equality.

> The one divergence from `==`: **NaN is equal to NaN**, matching Jest. `toBe` keeps `NaN ~= NaN`. Both agree `-0 == 0`.

### `toBeTruthy / toBeFalsy`

```luau
expect(0).toBeTruthy()
```

Luau truthiness: everything except `false` and `nil`.

> `0` and `''` are **truthy**, unlike in JavaScript.

### `toBeNil / toNotBeNil`

```luau
expect(map.missing).toBeNil()
```

Specifically `nil`, rather than merely falsy.

### `toBeA`

```luau
expect(workspace).toBeA('Instance')
```

A `typeof` check, so it understands Roblox types as well as Luau primitives.

### `toContain`

```luau
expect({ { id = 1 } }).toContain({ id = 1 })
```

For a string, a plain substring. For an array, any element **deeply equal** to the item — reference equality is rarely what a Luau test means.

### `toHaveLength`

```luau
expect({ 1, 2, 3 }).toHaveLength(3)
```

`#value`, for strings and tables.

> `#` is the array length, so a **dictionary table has length 0** — `expect({ a = 1, b = 2 }).toHaveLength(2)` fails saying its length is 0. Jest's `.toHaveLength` reads `.length` and throws on a plain object; here it is a quiet assertion failure instead.

### `toMatch`

```luau
expect(id).toMatch('^user_%d+$')
```

A **Luau string pattern** — not a substring, and not a regex.

> Use `toContain` when you want a literal. Both the value and the pattern must be strings.

### `toBeGreaterThan / toBeLessThan`

```luau
expect(elapsed).toBeLessThan(100)
```

With `…OrEqual` variants. Both sides must be two numbers or two strings.

### `toBeCloseTo`

```luau
expect(0.1 + 0.2).toBeCloseTo(0.3)
```

Passes when `|value - expected| < 10^-precision / 2`. `precision` defaults to `2`.

> NaN is never close to anything, including itself.

### `toThrow / toNotThrow`

```luau
expect(function () parse('{') end).toThrow('unexpected end')
```

The value must be a **function**, called in protected mode. With a substring, the message must contain it.

> `expect(parse('{'))` would raise while building the expectation, before any matcher ran — which is why a non-function is reported as matcher misuse.

### `toMatchSnapshot`

```luau
expect(render(doc)).toMatchSnapshot('compact')
```

The framework serializes; the CLI compares, diffs and updates. The optional hint distinguishes several snapshots in one test.

> Serialization is lossless with no depth cap, and a value that cannot serialize deterministically — a cycle, or a table or function key anywhere inside — fails the assertion rather than degrading the snapshot.

### `Chaining`

```luau
expect(n).toBeGreaterThanOrEqual(0).toBeLessThanOrEqual(100)
```

Every matcher returns the expectation, so several claims about one value read as one line.

> The first failing link reports; the links after it never run.

### `Matcher misuse`

```luau
expect(5).toMatch('%d')
```

Handing a matcher the wrong kind of thing fails as a clear misuse rather than a raw runtime error — which says your *test* is wrong, not the code under test.

## Backends

Where a suite's specs actually execute.

### `native`

```luau
backend = "native"
```

An embedded Luau VM inside the CLI. Pure logic; the default, the fast one, and the only one with coverage. A bare VM with **no task library**.

> On every backend a `print` in a test lands in the report where it ran. The spawned backends (`lune`, `lute`) frame their own events as sentinel-prefixed JSON lines on stdout and pass unprefixed output straight through, so a test cannot corrupt the stream; the sentinel is a framing device, not a security boundary — the code being framed is your own suite.

### `lune / lute`

```luau
backend = "lune"
```

A spawned `lune run` or `lute run` process, for `@lune/*` and `@lute/*` code. The real scheduler drives your test.

> On every backend a `print` in a test lands in the report where it ran. The spawned backends (`lune`, `lute`) frame their own events as sentinel-prefixed JSON lines on stdout and pass unprefixed output straight through, so a test cannot corrupt the stream; the sentinel is a framing device, not a security boundary — the code being framed is your own suite.

### `cloud`

```luau
backend = "cloud"
```

A real Roblox place through Open Cloud — Instances, services, the DataModel. Each task boots a fresh **game server**, so `IsServer()` is true and `IsClient()`/`IsStudio()` are false. For CI.

> **One Open Cloud task per spec file**, so each spec's events arrive already isolated and snapshots attribute correctly — at the cost of one round trip per spec, and of parallel CI jobs contending for the same place.

### `studio`

```luau
lest run engine --backend studio
```

A launched Roblox Studio, for the same engine code locally. Reached with `--backend studio`; the suite itself stays `backend = "cloud"` so CI keeps using cloud.

> Edit-mode context, at the command bar's permission level: `IsStudio()` is true and `IsClient()` **and** `IsServer()` are both true, so branch on `IsStudio()` and never on the other two. Every run pays a 15–45 s Studio boot; it refuses to run under `$CI` (exit 2) and is excluded from watch mode.

### `gargantuan`

```luau
[gargantuan]
binary = "vendor/gargantuan/build/gargantuan"
```

A spawned headless Gargantuan engine. **Experimental**: the engine has no tagged releases, so you build it from source and point `binary` at the result (without it, Lest looks for `gargantuan` on `PATH`).

> Its API surface is still filling in, so specs will hit `not yet implemented` edges — those are engine facts, not test failures. Excluded from watch mode and from `$CI` auto-enable; run it by naming the suite. Lest adapts to whatever stdio the engine build offers, probing each capability once per run: on a current build results travel through `ProcessService:WriteToStdout`, `FlushStdout` puts each on the wire as it happens and `ExitAsync(0)` ends the engine cleanly; older builds fall back to `print`, and Lest kills the engine a few seconds after the completion marker.

### `Isolation differs`

```luau
—
```

The registry resets before every spec file on every backend. Beyond that: `native` gives each file its own VM and `cloud` its own task, but `lune` and `lute` run a whole suite in one process over one module cache — so a helper module's state persists across spec files there.

## lest.toml

Every key is optional; the config's directory is the project root.

### `[suites.<name>]`

```luau
[suites.unit]
include = ["src/**/*.spec.luau"]
backend = "lute"
default = false
```

`include` is required. The table name is what `lest run <name>` takes and what the reporter labels the section with. Declaring no suite at all is valid: Lest synthesizes one named `specs` over `**/*.spec.luau`.

> **Hidden entries are never discovered** — dot-directories (`.git`, `.lest`) and dot-files (`.foo.spec.luau`) are skipped even when a glob matches them, because the watcher ignores them too and a spec that runs once and then never re-runs is worse than one that never runs.

### `default`

```luau
default = false
```

The suite runs only when named explicitly, **or when `$CI` is set** to anything but empty, `0` or `false`.

> `$CI` auto-enables it on every backend **except `studio` and `gargantuan`** — Studio is a GUI application that refuses to run in CI at all (exit 2 if you name its suite there), and gargantuan needs a locally built engine, so both are by-name-only. For a cloud suite the combination is what makes it bearable: out of your local loop, still gating every pull request.

### `[place]`

```luau
[place]
universe_id = 1234567890
place_id = 9876543210
file = "test-place.rbxl"
rojo = "default.project.json"
```

The Roblox place engine suites run in — `cloud` targets it through Open Cloud, `studio` launches it. The ids are not secret; the API key is read from `ROBLOX_API_KEY`, `LEST_API_KEY` or a `.env`.

### `[suites.<name>.place]`

```luau
[suites.engine.place]
universe_id = 1234567890
place_id = 9876543210
```

A per-suite place, overriding the top-level `[place]` **field by field** — for a project whose engine suites target different places.

### `[settings]`

```luau
[settings]
backend = "native"
timeout_ms = 5000
workers = 0
```

The default backend, the per-test budget, and native worker threads (`0` is one per CPU).

> There is a fourth key, `core`, pointing at a copy of the framework on disk. Leave it alone: the framework ships inside the binary and is written to `.lest/core` on demand, which is exactly what guarantees runner and framework are the same version. It exists so the Lest repository can dogfood its own working copy.

### `[studio]`

```luau
[studio]
executable = "D:/Custom/RobloxStudioBeta.exe"
```

One key, for a non-standard Studio install. By default Lest looks in the platform's install location (`%LOCALAPPDATA%\Roblox\Versions\…`, `/Applications`). The place it launches still comes from `[place]`.

### `[coverage]`

```luau
[coverage]
include = ["src/**"]
exclude = ["**/*.spec.luau", "**/*.spec.lua", "Packages/**"]
min = 80
```

Native suites only. `include` picks the candidates, `exclude` removes from what is left — **`exclude` wins** — and `min` fails the run below that percentage while turning measurement on.

> `exclude` **replaces** its defaults (the three patterns above) rather than adding to them, and setting `include` does not change them. `include` is unset by default; `include = []` is a config error (exit 2), because "cover nothing" and "cover everything" are equally fair readings of an empty list. Lest's own framework is never reported however wide `include` is.

### `Unrecognized keys`

```luau
Warning: Ignoring unrecognized key in lest.toml: bakcend
```

Tolerated so a config written for a newer Lest still parses — but named in a warning, because a typo'd key otherwise looks exactly like a working config.

### `Precedence`

```luau
CLI flag > suite setting > top-level default
```

`lest run unit --backend lune` ignores the suite's declared backend; `lest --min 90` overrides `[coverage] min`.

## CLI

### `lest`

```luau
lest
lest run unit scripts
lest --version
```

A bare `lest` is exactly `lest run`. With no arguments, every suite whose `default` is not `false` runs. Naming a suite twice runs it once, and an unknown name is an error listing the ones that exist.

> If *every* configured suite is `default = false` and you name none, that is a **tool error (exit 2)** telling you to name one — not a green run over nothing.

### `-t, --filter`

```luau
lest -t 'parser errors'
```

Run only tests whose full name contains the text — a plain substring, not a pattern.

> A filter that selects nothing is a **tool error (exit 2)**, not a green run — a typo'd `-t` in CI must not look like a passing suite.

### `--changed`

```luau
lest --changed origin/main
```

Diff against a git ref, then walk the inverted require graph for every spec whose transitive requires touched a changed file.

> Needs a git repository and a **valid ref**; anything else is a tool error (exit 2) — and most CI providers shallow-clone by default, which leaves `origin/main` missing, so fetch it (`fetch-depth: 0`). Nothing affected exits 0 with a note and skips any coverage gate. Conflicts with `--watch`, which is rejected rather than silently ignored.

### `--backend`

```luau
lest run engine --backend studio
lest run unit --backend lune
```

Force every selected suite onto one backend for this run: `native`, `lune`, `lute`, `cloud`, `studio` or `gargantuan`. A debugging override, and the documented way to reach the **studio** backend — an engine suite declares `backend = "cloud"` for CI and you name studio on the command line locally.

> Precedence is CLI flag > suite setting > top-level default. It is not a substitute for declaring the right backend in config.

### `--watch`

```luau
lest run unit -t 'parser' --watch
```

Graph-driven, not glob-driven: only specs whose transitive requires touched a saved file re-run.

> Debounced; hidden files ignored except `.luaurc` and `.config.luau`; `cloud`, `studio` and `gargantuan` always excluded. Saving `lest.toml` reloads it, and a bad reload is rejected while the previous config stays live.

### `--forbid-only`

```luau
lest --forbid-only
```

Exit 1 when any test was excluded by a focus modifier. A test-suite defect, not a tool error.

### `--reporter`

```luau
lest --reporter junit
```

`pretty` (default), `json` (one object per line), or `junit` (XML for CI annotations).

### `--no-color`

```luau
lest --no-color
```

Disable ANSI color. Already off automatically when the output stream is not a terminal, or when `NO_COLOR` is set.

### `--config`

```luau
lest --config packages/core/lest.toml
```

Path to the config file — it defaults to `./lest.toml` when present. The config file's **directory becomes the project root**, which every glob and relative path resolves against.

### `-u, --update`

```luau
lest -u
```

Overwrite snapshots that differ instead of failing on the difference.

### `--coverage / --min`

```luau
lest --min 80
lest --coverage-format lcov > lcov.info
```

Line coverage on native suites. `--min` implies `--coverage`; `lcov` writes the tracefile to stdout and moves the human report to stderr.

### `lest init`

```luau
lest init --yes
```

Detects a rojo project file, `lune`/`lute` on `PATH` and existing specs, then writes a commented config, an example spec and a `.luaurc` alias.

### `lest self`

```luau
lest self install
```

Copy into `~/.lest/bin` and add it to `PATH`. `uninstall` reverses it.

## Exit codes

Never conflated, which is the whole point.

### `0`

```luau
Everything passed
```

### `1`

```luau
Test failures
```

Including timeouts, spec load errors, a `--min` shortfall, and `--forbid-only`.

### `2`

```luau
Tool error
```

Bad config, a backend that could not start, an undecodable event stream, a filter or coverage gate over nothing.

> The run did not happen, so calling it a test failure would be a lie.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [lest](https://github.com/lest-luau/lest) | The test runner, framework included | 0.6.0 | `lpm tool add lest && lpm i` |
| [Lune](https://lune-org.github.io/docs) | Runtime for the `lune` backend | — | `lune must be on your PATH` |
| [Open Cloud](https://create.roblox.com/docs/cloud) | How `cloud` suites reach a real place | — | `ROBLOX_API_KEY=… (or a .env at the project root)` |
| [Lute](https://github.com/luau-lang/lute) | Runtime for the `lute` backend | — | `lute must be on your PATH` |
| [rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager that pins Lest per project | — | `rokit add lest-luau/lest` |
| [Luau LSP](https://github.com/JohnnyMorganz/luau-lsp) | Editor support for `--!strict` specs | — | `VS Code extension johnnymorganz.luau-lsp` |
| [Gargantuan](https://github.com/teamfireworks/gargantuan) | The experimental headless engine behind the `gargantuan` backend | pre-release, no tagged releases | `build from source; [gargantuan] binary = "…"` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`xit`.** Removed in 0.6; `it.skip` has the same signature.
- **`.not.toBe`.** `not` is a Luau keyword, so Jest's spelling does not exist here. The negations are `toNotBe`, `toNotEqual`, `toNotBeNil` and `toNotThrow`.
- **A bare top-level `backend` key.** That is the 0.3 spelling of `[settings] backend`; it warned in 0.4 and is removed in 0.5. The `[cloud]` table likewise became `[place]`.
- **`*` crossing `/` in `[coverage]` globs.** Through 0.4 it did, which made `src/*` and `src/**` identical. From 0.5 `*` stops at a directory boundary, so `vendor/*` must become `vendor/**` — a silent change that can make a passing `--min` gate start failing.
- **Expecting a mocked Instance.** Lest never emulates the engine. Engine code runs on the `cloud` or `studio` backend, in a real place.
