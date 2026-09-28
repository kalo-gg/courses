# 5. Space

*Positions and directions, orientation with CFrame, the difference between object and world space, and moving whole models.*

Roblox Fundamentals · Medium

---

Everything in the world sits somewhere and faces some way, and this module is the math for both. It starts with Vector3: points, offsets and directions, and the few operations that answer how far and whether something is in front. CFrame adds orientation, built with `lookAt` and `Angles`, and the order of multiplication separates a part's own space from the world's. It ends with models: moving a whole build by its pivot with `PivotTo`, without pulling it apart.

## Lessons

1. [Vector3](01-vectors.md) — One type for points, offsets and directions, and the operations that answer how far and which way.
2. [CFrame](02-cframes.md) — A position plus an orientation: build one with `lookAt` or `Angles`, and read which way it faces.
3. [Object and world space](03-object-and-world-space.md) — `cf * offset` measures the offset from the part itself: in front, above or to the right, whichever way it faces.
4. [Models and pivots](04-models-and-pivots.md) — Move a whole model by its pivot with `PivotTo`, and read where it is with `GetPivot` and `GetBoundingBox`.
