# 6. Best Practices & Capstone

*Keeping a dependency tree boring, choosing between Wally and pesde honestly, and shipping a library.*

Wally · Hard

---

The habits that keep a dependency tree boring, and an honest answer to the question every Roblox project now asks: Wally or pesde. This module commits and reads the lockfile, weighs the two package managers on numbers rather than taste, puts the whole install in CI, and audits a package before depending on it. Then the capstone, where you publish a library and consume it from the place you built earlier.

## Lessons

1. [Keeping the tree boring](01-lockfile-hygiene.md) — Commit the lockfile, read its diff, pin what is expensive to break, and delete it when it wedges.
2. [Wally or pesde](02-wally-or-pesde.md) — Two real answers: adoption is Wally's, development is pesde's, and each carries a named risk.
3. [Continuous integration](03-ci.md) — A workflow that installs, restores types, builds and publishes, and notices when a publish fails.
4. [Auditing a dependency](04-auditing.md) — Read the index entry, the archive and the realm before a package becomes yours to maintain.
