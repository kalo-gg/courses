# 6. Best Practices & Capstone

*The messages that mean something else, the lockfile habits, the Wally migration, and shipping Loot.*

pesde · Hard

---

Troubleshooting reads pesde's messages for what they really mean; The lockfile and reproducible installs covers what pesde.lock records and which commands touch it; Migrating a Wally project maps wally.toml onto pesde.toml and names the three things that break; Designing a publishable library plans the capstone's tree. The module ends with Ship Loot: the manifest made publishable and a test place whose built tree is checked.

## Lessons

1. [Troubleshooting resolution and linking](01-troubleshooting.md) — The messages pesde prints when something is wrong, what each one really means, and the fix for each.
2. [The lockfile and reproducible installs](02-lockfile-hygiene.md) — What pesde.lock records, which commands touch it, and the habits that keep every machine installing one tree.
3. [Migrating a Wally project](03-migrating-from-wally.md) — wally.toml to pesde.toml, Packages to roblox_packages, and the three things that break on the way.
4. [Designing a publishable library](04-library-architecture.md) — Laying out a Roblox library so it publishes cleanly, tests in Studio and can serve two targets.
