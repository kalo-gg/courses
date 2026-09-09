# Six places a test can run

*One embedded VM, two spawned runtimes, and three real engines, and how to pick the cheapest one that is true.*

Lest · Backends · 11 min

---

A suite's **backend** is where its specs actually execute. Six values are on offer, and choosing between them is one decision made once: the cheapest environment in which the test you are about to write can be true.

| Backend | Runs in | Coverage | Watch |
| --- | --- | --- | --- |
| `native` | An embedded Luau VM inside the CLI | ✅ | ✅ |
| `lune` | A spawned `lune run` process | — | ✅ |
| `lute` | A spawned `lute run` process | — | ✅ |
| `cloud` | A real Roblox place via Open Cloud | — | — |
| `studio` | A launched Roblox Studio | — | — |
| `gargantuan` | A headless Gargantuan engine | — | — |

**Choosing per suite**

```toml
[suites.unit]
include = ["src/**/*.spec.luau"]        # native by default

[suites.scripts]
include = ["tools/**/*.spec.luau"]
backend = "lune"                        # these specs use @lune/*
```

## native

The default and the fast one. The CLI embeds a Luau VM and runs specs on a worker pool — one worker per CPU, **one fresh VM per spec file**, nothing shared. It is the only backend with line coverage, because the VM exposes the hooks, and the only one fast enough for watch mode.

> **It has no `@lune/*`, no `@lute/*`, and no engine APIs**
>
> A spec that requires one fails with a message **naming the backend that has it**, rather than a confusing module-resolution error. That is the whole philosophy in one diagnostic: the tool knows which environment you needed and tells you where to find it.

## lune and lute

One abstraction, two runtimes — the framework is pure Luau, so it runs unmodified in both. The CLI generates a harness into `.lest/`, spawns `lune run` or `lute run` on it, and decodes the events it prints back into the same results bus.

The costs are real: no coverage, because the VM hooks do not cross a process boundary; a timeout that is a whole-suite budget enforced by killing the process; and a runtime that has to be installed — Lest checks your `PATH` and prints the install command.

**A spec that needs a runtime**

```luau
local fs = require('@lune/fs')       -- resolves on lune; on native the failure names lune

it('reads the manifest', function ()
	expect(fs.readFile('wally.toml')).toContain('[package]')
end)
```

Not runnable here — the sandbox is a bare VM like native, which is the point.

## cloud

For code that touches real engine APIs there is no faking it, so Lest does not try. The `cloud` backend bundles your specs and the framework into one script and submits it as an Open Cloud **Luau execution task** against a real place. Each task boots a fresh Roblox game server and runs the bundle as an ordinary server script, so `RunService:IsServer()` is true and server-only services behave as in production.

> **One task per spec file**
>
> Not one task per run. Each spec file gets its own Open Cloud task, which is why the next lesson can promise a cloud spec fresh module state and correctly attributed snapshots — and why cloud is slow: **one network round trip per spec**. Each task's in-engine deadline comes from `timeout_ms` for that one file, so it does not grow with the size of the suite.

**Key points**

- `native`: embedded VM, one per spec file, the only backend with coverage.
- `lune`/`lute`: a spawned real runtime, harness in `.lest/`, no coverage.
- `cloud`: a real place through Open Cloud, one task per spec file, for CI.
- A spec that requires what a backend lacks fails naming the backend that has it.
- Pick the cheapest backend that makes the test true.
