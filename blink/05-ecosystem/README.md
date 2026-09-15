# 5. Tooling & Ecosystem

*The Studio plugin, a CI job that does not lie to you, the Zap question, and exports for ECS replication.*

Blink · Hard

---

Blink is a compiler, and a compiler lives inside other tools. This module covers the Studio plugin — the same compiler, without a terminal — a CI job that actually fails when the description is broken, the honest comparison with Zap that every Roblox project ends up making, and exports, which turn a Blink type into a pair of buffer functions an ECS can use.

## Lessons

1. [The Studio plugin](01-the-studio-plugin.md) — The same compiler inside Studio, for people who would rather not open a terminal.
2. [Blink in CI](02-blink-in-ci.md) — The compiler exits 0 on every error, so a CI job has to read the output instead.
3. [Blink or Zap](03-blink-or-zap.md) — Two IDLs with the same idea, and the differences that actually decide it.
4. [Exports and ECS replication](04-exports-and-ecs.md) — export gives a type its own Read and Write, which is how a component gets replicated.
