# 5. Scale

*Streaming the world in, running on more than one core, and measuring before you optimise.*

Roblox Advanced · Hard

---

A game that runs well with ten players and a small map can stutter with fifty players and a big one: the world is too large to send to every device, NPC logic fills one core while the others wait, and frames run long for reasons nobody has measured. This module tunes instance streaming from Studio and the server, including models that stay loaded for one player and areas fetched before a player arrives. Then it spreads work over several cores with Parallel Luau: Actors, the parallel and serial phases, thread safety, and the messages and SharedTables that carry data between Actors. It ends with profiling, the MicroProfiler, its labels and the Developer Console, so you measure before you optimise.

## Lessons

1. [Instance streaming](01-instance-streaming.md) — Tune streaming for the whole place, keep one model loaded for the player who needs it, and fetch an area before moving a player there.
2. [Parallel Luau](02-parallel-luau.md) — Put scripts under Actors, read the world in the parallel phase, and change it only after task.synchronize.
3. [Sharing between actors](03-sharing-between-actors.md) — Send an Actor a message, which copies its arguments, or share a SharedTable, which every Actor sees at once.
4. [Profiling](04-profiling.md) — Find where the frame goes with the MicroProfiler and your own labels, and read the Developer Console's other tools, before changing anything.
