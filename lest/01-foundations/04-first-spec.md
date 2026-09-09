# Your first spec

*Write a spec, run it, read a passing report, and then read a failing one.*

Lest · Lest Foundations · 11 min

---

Every spec file has the same shape: require the framework, pull the API into locals, declare tests, `return nil`. The return value is never read — Lest collects tests through registration — but the file is a module, so it needs one.

**Running it**

```bash
lest                # every default suite
lest run unit       # one suite
lest -t 'cart'      # only tests whose full name contains "cart"
lest --watch        # re-run affected specs on save
```

`-t` filters on the **full name** — the describe path and the test's own name joined by single spaces — and `lest run unit` names a suite. Both come back in *Writing Tests*; for now, a bare `lest` is all you need.

**The report**

```text
specs (native)
  cart
    ✓ sums line items (0.0ms)
    ✓ rejects a negative quantity (0.1ms)

Slowest Tests:
  cart › rejects a negative quantity (0.1ms)
  cart › sums line items (0.0ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        0.01s
```

`describe` blocks become indented headers, and the suite is labeled with the environment it ran in — `specs` here because there is no `lest.toml`; with `[suites.unit]` declared the same run opens `unit (native)`. The `Slowest Tests:` block is the one place the ` › ` glyph appears.

## When a test fails

A failing test prints `✗` in the tree, and after the tree one block per failure: `●` and the full name, the matcher's message, then the two values it compared. That pair is the point of using a matcher — `Expected: 150 / Received: 200` says more than a bare `assert` ever could. The summary then counts `1 failed`, and the process exits **1**: tests failed. Exit **2** is reserved for Lest itself breaking, and *Continuous integration* is built on that distinction.

**A failing report**

```text
specs (native)
  cart
    ✓ sums line items
    ✗ applies the discount

● cart applies the discount

    Values are not identical
    Expected: 150
    Received: 200

Test Suites: 1 failed, 0 passed, 1 total
Tests:       1 failed, 1 passed, 2 total
Snapshots:   0 total
```

`Expected` is what you passed to the matcher; `Received` is what `expect` was given — the value your code produced.

> **The exercises below really run**
>
> Kalo's sandbox implements the Lest framework in Luau — `describe`, `it`, `expect`, the hooks and every matcher — and runs your tests the way the CLI does: your file registers, then the tests execute. What you see under the editor is a real report of your real tests.
>
> Two things the browser cannot do. There is no disk, so `require` of your own files is unavailable: **the module each exercise tests is already defined for you** (`total`, `lineOf`, `withdraw` and friends arrive that way), and `require('./cart')` reaches a small stand-in so this lesson's sample runs as written. And there is no clock, so the sandbox prints no `(0.0ms)` timings, no `Slowest Tests:` block and no `Time:` line — everything else above is what you will see.

**Key points**

- A spec requires `@lest`, pulls `describe`/`it`/`expect` into locals, registers tests, and ends with `return nil`.
- `lest` runs every default suite; `lest run <suite>` one; `-t` filters by full name.
- `✓`/`✗` in the tree, then a `●` block per failure with Expected and Received.
- Exit 1 means tests failed; exit 2 means Lest could not run them.
