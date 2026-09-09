# 6. Larvae for a Team

*The editor server, the CI gates, the pipeline you are replacing, and one config that builds everything.*

Larvae · Hard

---

Running larvae alone is one thing; running it for a team is another. This module covers the [lsp] table and the second binary behind it, the four commands a CI job runs and which of them can fail, how a darklua, stylua and selene pipeline maps onto larvae name by name, and how to lay out one config — a base, a leaf, overrides and profiles — so every build of Lantern comes from the same file. The final project ships that toolchain.

## Lessons

1. [The editor server](01-the-editor.md) — The lsp table: what the server answers, which files it claims, and where the types come from.
2. [larvae in CI](02-larvae-in-ci.md) — Four commands that write nothing, which of them can fail, and a workflow that installs larvae and runs them.
3. [Migrating from darklua, stylua and selene](03-migrating-a-pipeline.md) — darklua, stylua and selene map onto larvae name by name; the three rules that moved and the two files that become one.
4. [One config, many builds](04-one-config-many-builds.md) — A base, a leaf, overrides and profiles: the layout that gives every build of Lantern one source of truth.
