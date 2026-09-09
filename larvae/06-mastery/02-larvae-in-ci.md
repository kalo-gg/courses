# larvae in CI

*Four commands that write nothing, which of them can fail, and a workflow that installs larvae and runs them.*

Larvae · Larvae for a Team · 10 min

---

CI is the four commands that write nothing, in the order that fails fastest. `larvae fmt --check` exits non-zero if any file would change. `larvae lint` exits non-zero only on a `deny`. `larvae check` fails on an unresolvable require or a realm violation whatever the levels say, and on whichever of `cycles`, `unused_modules` and `early_require` the project set to `deny`. `larvae self sync-luaurc --check` fails when `.luaurc` has drifted from `[aliases]`. Then, and only then, the build.

**The gates**

```bash
larvae worm install                # the worms larvae.toml names are not in the checkout
larvae fmt --check                 # would any file change?
larvae lint                        # any deny?
larvae check                       # any require that cannot work, or a denied cycle?
larvae self sync-luaurc --check    # do .luaurc and [aliases] still agree?
larvae process --profile release   # the build, after every gate
```

One command in the job does download: `larvae worm install`, because the worms `larvae.toml` names are not in the checkout, and a `.luaux` file with no worm is a file larvae cannot lint or format. It comes first. What CI must never run is anything that writes to the tree — `larvae fmt` without `--check`, or `larvae init` — because a job that changes files is a job whose green means nothing.

## Installing larvae in a job

The job needs the binary, and the analyzer only if it runs `larvae analyze`. `cargo install larvae` works everywhere and is slow; a tool manager with a pinned manifest is the usual route, and a `rokit.toml` that pins `larvae` is what Lantern commits so every machine and every job runs the same release. Whichever installs it, `larvae self install` is what fetches `larvae-lsp` beside it when a job wants types.

**rokit.toml**

```toml
[tools]
larvae = "larvae-luau/larvae@0.8.1"
```

**.github/workflows/ci.yml**

```text
name: ci
on: [push, pull_request]
jobs:
  larvae:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: CompeyDev/setup-rokit@v0.2.1   # installs what rokit.toml pins
      - run: larvae worm install
      - run: larvae fmt --check
      - run: larvae lint
      - run: larvae check
      - run: larvae self sync-luaurc --check
      - run: larvae process --profile release
```

> **Pin it, or the diff moves under you**
>
> A formatter that updates between two runs changes what `--check` accepts, and a linter that gains a lint turns a green job red with nobody's commit to blame. Pin the release in the manifest and bump it on purpose. `larvae self update` is for your machine, not for the job.

**Key points**

- Four gates write nothing: `fmt --check`, `lint`, `check`, `self sync-luaurc --check`; only `deny`, a would-change, a drift, or a require that cannot work fails.
- `larvae worm install` comes first in a job: claimed files need their worm.
- Pin the release in the tool manager's manifest; `self update` is for your machine.
- `larvae process --profile release` is the build step, after every gate.
