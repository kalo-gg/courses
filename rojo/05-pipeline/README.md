# 5. Packages, Syncback & CI

*Wally packages behind optional paths, Studio work pulled back into git, CI that rebuilds, and publishing.*

Rojo · Medium

---

A repository is more than your own files. This module brings in Wally packages and explains why your project settings stop at the package boundary, pulls a builder's Studio work back into files with syncback and its rules, decides what git commits and what a CI job rebuilds on every push, and publishes the result: a place through Open Cloud with rojo upload, or a Studio plugin with rojo build --plugin.

## Lessons

1. [Packages with Wally](01-wally-packages.md) — Declare dependencies in wally.toml, install them into Packages folders, and mount those folders behind optional paths.
2. [Syncback: adopting a Studio place](02-syncback.md) — Pull a saved place back into files with rojo syncback, limit what comes back with rules, and adopt an existing game.
3. [Git and CI](03-git-and-ci.md) — What to commit, what to ignore, and a GitHub Actions job that builds the place on every push.
4. [Publishing places and plugins](04-publishing.md) — Push a built place to Roblox with rojo upload, and build a Studio plugin straight into the plugins folder.
