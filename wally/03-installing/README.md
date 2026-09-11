# 3. Installing and the Tree

*What install writes to disk, what the lockfile really says, and why the resolver refuses to backtrack.*

Wally · Medium

---

`wally install` reads the manifest, resolves a version for every package in the graph, writes a lockfile and builds three directories of link modules. Each of those steps has a behaviour worth knowing before it surprises you, and the resolver in particular does something Cargo does not. This module ends with the place file that consumes them.

## Lessons

1. [wally install](01-installing.md) — One command reads the manifest, wipes three directories and writes them again from scratch.
2. [What lands on disk](02-the-package-tree.md) — Three directories of one-line link modules sitting over a flat index of every resolved package.
3. [wally.lock](03-the-lockfile.md) — The lockfile is short, oddly literal, and nothing like the example in the README.
4. [How a version gets chosen](04-resolution.md) — Greedy, first-fit, and no backtracking, which is why a stale lockfile can wedge an install.
