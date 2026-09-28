# 6. The world

*Asking the world what is there, finding a way through it, and bringing characters to life.*

Roblox Intermediate · Medium

---

A game keeps asking the world questions: what is under the crosshair, who is inside the blast, how does this guard get round the wall. This module answers them. It starts with `workspace:Raycast` and the `RaycastParams` lists that decide what a ray may hit, then sweeps whole shapes with `Spherecast` and gathers everything inside a volume with the overlap queries and `OverlapParams`. PathfindingService then computes a route that an NPC walks with `Humanoid:MoveTo`, computing it again when something blocks the way. It ends with characters that move and make noise: animation tracks loaded through the Animator, and sound built from the Audio API's players, wires and outputs.

## Lessons

1. [Raycasting](01-raycasting.md) — Ask what lies along a line with `workspace:Raycast`, and decide what it may hit with `RaycastParams`.
2. [Shapecasts and overlaps](02-shapecasts-and-overlaps.md) — Gather every part inside a volume, or sweep a whole shape along a path, and turn parts back into characters.
3. [Pathfinding](03-pathfinding.md) — Compute a route round obstacles, walk an NPC along it with `MoveTo`, and compute again when it is blocked.
4. [Animation and sound](04-animation-and-sound.md) — Play animation tracks through the Animator, and build sound from the Audio API's players, wires and outputs.
