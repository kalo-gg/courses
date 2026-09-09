# 7. The JECS Ecosystem

*The tools around JECS: the Jabby inspector, the Planck scheduler, replecs replication, roblox-ts, and moving from Matter.*

JECS · Hard

---

JECS ships a small core; inspection, scheduling and replication come from the ecosystem. This module installs and uses the packages that pair with JECS 0.11 — Jabby, Planck, replecs and the buffer-based networking libraries — shows the TypeScript typings, and gives this course's own Matter to JECS mapping. Versions were checked in September 2026; confirm against your lockfile.

## Lessons

1. [Debugging with Jabby](01-debugging-with-jabby.md) — Install the Jabby inspector, register your world and systems, query entities in-game, and know the fallback.
2. [Scheduling with Planck](02-planck-scheduler.md) — Phases, pipelines and run conditions for running JECS systems with the Planck scheduler.
3. [Replication with replecs](03-replication-with-replecs.md) — Mark components and entities for replication, pick reliable or unreliable channels, and carry the buffers.
4. [roblox-ts and coming from Matter](04-roblox-ts-and-matter.md) — Using JECS from TypeScript with @rbxts/jecs and its companions, plus this course's own Matter to JECS mapping.
