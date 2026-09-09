# Planning a suite

*Which suite each module belongs in, a coverage number you can hold, and a plan you can run as todos.*

Lest · Suite Design & Capstone · 11 min

---

Before the capstone, the plan. The shop has three modules: `cart` (pure functions, tested in Project 1), `checkout` (pure logic with two injected collaborators), and `orders` (writes to a DataStore). The first two belong in a **unit suite on native**: fast, covered, under watch. The third touches the engine and belongs in an **engine suite on cloud**, `default = false`, so it runs on every pull request and never in the local loop.

**The layout**

```text
shop/
  cart.luau          cart.spec.luau          unit    (native)
  checkout.luau      checkout.spec.luau      unit    (native)
  orders.luau        engine/orders.spec.luau engine  (cloud, default = false)
  lest.toml          default.project.json    .github/workflows/test.yml
```

Specs beside the module they test, engine specs in their own tree so the two suites' globs cannot overlap. A require resolves against the requiring file's own directory, so `checkout.spec.luau` says `require('./cart')` while `engine/orders.spec.luau` says `require('../orders')` — the capstone's tree is flat precisely because that is the one layout where the sandbox, which resolves from the project root, agrees with the CLI.

## A coverage number you can hold

Coverage is a floor, not a target. Measure once, set `min` a few points under what you have, and raise it as the suite grows: a gate you can hold on every pull request beats a 100% you reach once. Count only what you own — `exclude` the specs, listing the defaults again since setting it replaces them — and remember that a `--changed` run that affects nothing skips the gate, and that the engine suite contributes nothing to it because coverage is native-only.

| Module | Suite | Backend | In the coverage table |
| --- | --- | --- | --- |
| `cart` | unit | native | yes |
| `checkout` | unit | native | yes |
| `orders` | engine | cloud | no — coverage is native-only |
| the workflow | both invocations | — | the gate runs on the narrowed native pass |

**The layout, as configuration**

```toml
[suites.unit]
include = ["*.spec.luau"]              # cart and checkout, on native

[suites.engine]
include = ["engine/**/*.spec.luau"]
backend = "cloud"
default = false                        # named locally, automatic under $CI
```

Two suites, two globs that cannot overlap. `[place]` and `[coverage]` are the two blocks this is still missing — the capstone adds them.

Write the plan as a spec. `it.todo` registers a name with no body and reports it as skipped, so a `describe` full of todos runs green, prints as a checklist, and turns into the suite one body at a time — with no separate document to fall out of date.

**Key points**

- Pure logic and faked boundaries: the unit suite on native. Engine calls: an engine suite on cloud, `default = false`.
- Specs beside modules; engine specs in their own tree.
- Set `min` where you can hold it and ratchet up; coverage is native-only.
- A plan written as `it.todo`s is a plan that runs.
