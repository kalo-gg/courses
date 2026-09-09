# 5. Workspaces, Tooling & CI

*Many packages in one repository, dependencies bent to fit, packages you run, and a pipeline that trusts the lockfile.*

pesde · Hard

---

The features you reach for once a project stops being one package. Workspaces holds a monorepo together with sigils that rewrite themselves; Overrides and patches redirect a transitive dependency or edit one in place; Binaries, scripts and pesde run covers packages you execute and how run resolves its argument; Engines and CI pins the runtimes a project needs and lays out the install, test and publish steps a pipeline should run.

## Lessons

1. [Workspaces](01-workspaces.md) — Several packages in one repository, commands that run on all of them, and four sigils that rewrite themselves.
2. [Overrides and patches](02-overrides-and-patches.md) — When a dependency's dependency is wrong you override it; when the dependency itself is, you patch it.
3. [Binaries, scripts and pesde run](03-binaries-and-scripts.md) — Packages you run rather than require, one-off execution with pesde x, and how pesde run resolves its argument.
4. [Engines and CI](04-engines-and-ci.md) — Pinning the runtimes a project needs, and the install, test and publish steps a pipeline should run.
