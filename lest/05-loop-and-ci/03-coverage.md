# Coverage

*Measure line coverage on the native backend, get a clean lcov file out, and gate a run on a minimum.*

Lest · The Loop and CI · 11 min

---

Coverage is the share of lines the suite executed. Lest measures it on **native suites only** — the VM exposes the hooks, and they do not cross a process boundary — which is one more reason pure logic belongs there. A number on its own is decoration; a gate that fails the run below it is what makes the number mean something.

**Measuring and gating**

```bash
lest --coverage
lest --min 80
lest --coverage-format lcov > lcov.info
```

Native suites only — the VM hooks do not cross a process boundary. `--min` implies `--coverage`.

> **Why the lcov redirect is clean**
>
> Under `--coverage-format lcov` the tracefile owns **stdout** and the human report moves to **stderr**, so `> lcov.info` captures a tracefile and nothing else. You still see the test report in your terminal.

## What counts

> **`[coverage] exclude` replaces the defaults**
>
> It does not add to them. The defaults are `["**/*.spec.luau", "**/*.spec.lua", "Packages/**"]`, so if you set `exclude` yourself and still want spec files out of the table, list them again. And since 0.5, `*` stops at a directory boundary — `vendor/*` covers one level, `vendor/**` covers the tree.

Subtracting is not the only move. When only one tree counts — a library where coverage means `src/` and nothing else — **`include`** names what you want instead, and the two compose with `exclude` winning: `include` picks the candidates, `exclude` removes from what is left. Setting `include` does *not* drop the default excludes, which are defaulted independently, so spec files stay out either way. And Lest's own framework is never reported, however wide you make it.

**A tree minus its generated corner**

```toml
[coverage]
include = ["src/**"]
exclude = ["**/*.spec.luau", "src/generated/**"]
```

`include = []` is a **config error (exit 2)**, not "cover nothing": an empty list reads equally well as "cover everything", so Lest refuses to guess. Remove the key if you meant no narrowing.

`--min` implies `--coverage`, and `lest --min 90` on the command line overrides `[coverage] min`. Two edges: a `--changed` run that affects nothing skips the gate explicitly, and a coverage minimum over an empty table — no native suite ran — is a tool error rather than a pass, because "nothing measured" is not "100%".

**Key points**

- Native suites only; `--min` implies `--coverage`.
- `lcov` owns stdout and the human report moves to stderr, so `> lcov.info` is clean.
- `exclude` replaces its defaults; `include` narrows and does not touch them; `exclude` wins.
- `include = []` and a gate over nothing are exit 2.
