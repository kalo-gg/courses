# Suites and lest.toml

*Zero config runs a suite named specs; lest.toml names your own, picks a backend and sets the defaults.*

Lest · Lest Foundations · 10 min

---

A **suite** is a set of spec files and the backend they run on. With no `lest.toml` at all, Lest synthesizes one `native` suite named `specs` over `**/*.spec.luau` — for many projects that is genuinely enough, and you can write a spec and run `lest` without configuring anything. The moment you declare a suite of your own, the synthesized one goes away.

**A real config for a real project**

```toml
[suites.unit]
include = ["src/**/*.spec.luau"]
```

Two lines. Everything else has a default worth having.

## What a suite can say

| Key | Meaning | Default |
| --- | --- | --- |
| `include` | A list of globs selecting the suite's specs | required |
| `backend` | `native`, `lune`, `lute`, `cloud`, `studio` or `gargantuan` | `[settings] backend`, else `native` |
| `default` | `false` runs the suite only when named or when `$CI` is set | `true` |

The table name is what `lest run <name>` takes and what the report labels the section with — `unit (native)` instead of `specs (native)`.

Every key is optional except `include`. Hidden entries are never discovered: dot-directories like `.git` and `.lest` and dot-files like `.foo.spec.luau` are skipped even when a glob matches them, because the watcher ignores them too, and a spec that runs once and then never re-runs is worse than one that never runs.

**lest.toml**

```toml
[settings]
backend = "native"   # for suites that do not say otherwise
timeout_ms = 5000    # the per-test budget
workers = 0          # native worker threads; 0 is one per CPU

[suites.unit]
include = ["src/**/*.spec.luau"]

[suites.scripts]
include = ["tools/**/*.spec.luau"]
backend = "lute"
```

Precedence is **CLI flag > suite setting > top-level default**: `lest run scripts --backend lune` ignores the suite's `lute` for that one run.

> **Typos are warnings, not errors**
>
> Unrecognized keys are tolerated so a config written for a newer Lest still parses, but each one is named in a warning: `Warning: Ignoring unrecognized key in lest.toml: bakcend`. Read the first lines of output after you edit the config; a typo'd key otherwise looks exactly like a working one.

The config file's directory is the project root: every glob and relative path resolves against it, and `lest --config packages/core/lest.toml` moves the root along with the file. There is a fourth `[settings]` key, `core`, that points at a framework on disk; leave it alone — it exists so the Lest repository can test its own working copy.

**Key points**

- No config means one `native` suite named `specs` over `**/*.spec.luau`.
- A suite is `[suites.<name>]` with `include`; `backend` and `default` are optional.
- `[settings]` holds the defaults: `backend`, `timeout_ms`, `workers`.
- CLI flag > suite setting > top-level default.
- Dot-files and dot-directories are never discovered.
