# What is Lest?

*A toolchain rather than a library, a position on mocking, and six places a test can run.*

Lest · Lest Foundations · 9 min

---

Most Luau test frameworks are packages you install. Lest is a **single binary that carries the framework inside it** — you install one thing, and the runner and the framework are by construction the same version. There is no protocol between them to get out of step, and nothing to add to your dependency list.

The API is Jest-shaped, so most of it will look familiar:

**cart.spec.luau**

```luau
--!strict
local Lest = require('@lest')
local describe, it, expect = Lest.describe, Lest.it, Lest.expect

local cart = require('./cart')

describe('cart', function ()
	it('sums line items', function ()
		expect(cart.total({ 3, 4 })).toBe(7)
	end)
end)

return nil
```

## No engine emulation. Ever.

> **The position that shapes everything else**
>
> Lest mocks no Instances and fakes no runtime, on principle: **partial mocks produce confident wrong tests.** A fake `Instance` that is 90% right gives you a green suite and a broken game. So if a test needs an environment, Lest runs it in that environment — really.

| Backend | Runs in | For |
| --- | --- | --- |
| `native` | An embedded Luau VM inside the CLI | Pure logic. The default, and the fast one. |
| `lune` | A spawned `lune run` process | Scripts using `@lune/*`. |
| `lute` | A spawned `lute run` process | `@lute/*`, code transforms, tooling. |
| `cloud` | A real Roblox place via Open Cloud | Instances, services, the DataModel — in CI. |
| `studio` | A launched Roblox Studio | The same engine code, locally. |
| `gargantuan` | A headless Gargantuan engine | Experimental. |

Six of them, one merged report. *Backends* takes them apart; for now, `native` is the one you will live in.

You write `describe` / `it` / `expect` once. Where a suite runs is a line of configuration, and everything downstream — reporters, snapshots, filters, CI output — neither knows nor cares where a test executed. A single run can span several suites on several backends and still print one merged report.

**lest.toml**

```toml
[suites.engine]
include = ["tests/engine/**/*.spec.luau"]
backend = "cloud"
```

The spec above does not change to move between the six. This does — one key, and *Backends* is the module about choosing its value.

**Key points**

- One binary carries the framework; runner and framework are always the same version.
- The API is Jest-shaped: `describe`, `it`, `expect`.
- Lest never emulates the engine; engine code runs in a real engine.
- Where a suite runs is one line of config, and nothing downstream cares.
