# 3. Roblox Projects

*Sync config generators, a test place, and the one habit you have to unlearn from Wally.*

pesde · Medium

---

Roblox is the target that needs help: pesde installs files, and something must tell your sync tool what to do with them. The sync config generator sets that script up; Unlearning Wally explains the project file a pesde package must not have and the tree that replaces it; A test place for Studio puts the project file where it belongs; Wally dependencies wires up the sourcemap generator, [place] and roblox_server. It closes with Loot's manifest.

## Lessons

1. [The sync config generator](01-sync-config.md) — Why Roblox needs a second step, the scripts package that provides it, and the install that makes it real.
2. [Unlearning Wally](02-unlearning-wally.md) — The project file a pesde package must not have, what the consumer's generator writes instead, and the tree that results.
3. [A test place for Studio](03-test-place.md) — Where the project file does belong: a place that mounts your package and its dependencies for Studio and Luau LSP.
4. [Wally dependencies and [place]](04-wally-dependencies.md) — A Wally package needs the sourcemap generator to get an entry point; [place] and roblox_server round out the setup.
