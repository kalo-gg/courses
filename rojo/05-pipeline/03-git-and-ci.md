# Git and CI

*What to commit, what to ignore, and a GitHub Actions job that builds the place on every push.*

Rojo · Packages, Syncback & CI · 10 min

---

The point of files was always version control. A Rojo repository commits the project file, `src/`, `rokit.toml`, `wally.toml` and `wally.lock`, and ignores what tools generate: built places, `sourcemap.json`, Studio's `.rbxl.lock` files and Wally's `Packages/` output (`wally install` recreates it from the lockfile).

**.gitignore**

```text
# built places and models
/*.rbxl
/*.rbxlx
/*.rbxm
/*.rbxlx.lock
/*.rbxl.lock

# generated
sourcemap.json
/Packages
/ServerPackages
```

## A build on every push

Continuous integration (CI: a service that runs commands on every push) then does what a teammate would: install the pinned tools, install packages, build. Because Rokit reads `rokit.toml`, the same versions run everywhere. The community `CompeyDev/setup-rokit` action installs Rokit and every tool the file lists.

**.github/workflows/build.yml (minimal)**

```text
name: Build place
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: CompeyDev/setup-rokit@v0.2.1
      - run: rojo build -o build.rbxl
```

`setup-rokit` reads `rokit.toml` and installs every pinned tool, so the job runs the same Rojo a teammate does.

That job is missing two things the exercise adds. Without `wally install` before the build, the optional `Packages` mount is skipped in silence and the place ships with `ReplicatedStorage.Packages` absent — the build passes, the game breaks at runtime. And without an `upload-artifact` step nothing is kept: the `.rbxl` is built on a runner that is thrown away. A publishing step with `rojo upload` goes after the build on merges to `main`, with the Open Cloud key stored as a repository secret; Publishing places and plugins covers the command.

> **Secrets, not cookies**
>
> CI should authenticate with an Open Cloud API key kept as a repository secret and passed as an environment variable. Never commit a `.ROBLOSECURITY` cookie, and never put the key in the workflow file.

**Key points**

- Commit the project file, `src/`, `rokit.toml`, `wally.toml` and `wally.lock`; ignore builds, `sourcemap.json`, lock files and `Packages/`.
- CI installs tools with Rokit, runs `wally install`, then `rojo build`, and keeps the place as an artifact; publishing is a separate step with a secret.
- One project file per place lets CI build each artifact from the same source tree.
