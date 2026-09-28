# 6. Motion

*The frame loop, tweens, physics and assemblies, and constraints that move things properly.*

Roblox Fundamentals · Medium

---

Until now everything you built stood still. This module makes it move. It starts with the frame: the RunService events the engine fires every frame, in order, and motion scaled by the frame's length so it runs at the same speed at any frame rate, plus the rest of the task library. Tweens handle any change with a start, an end and a duration. Then physics: anchored and simulated parts, assemblies welded into one body, the collision switches on every part and collision groups on Workspace. It ends with constraints, the attachments, hinges and sliders that let physics drive a part, which is how the obby's spinners and moving platforms are built.

## Lessons

1. [The frame](01-the-frame.md) — RunService fires events every frame; scale what you move by the frame's length and it runs at the same speed at any frame rate.
2. [TweenService](02-tweens.md) — Give TweenService a goal and a TweenInfo, and it moves the properties there along an easing curve.
3. [Physics and assemblies](03-physics-and-assemblies.md) — Anchored parts stay where scripts put them; the rest are simulated in assemblies of welded parts, filtered by collision switches and groups.
4. [Constraints](04-constraints.md) — Attachments mark the joint, a constraint says how the parts may move, and an actuator lets physics drive them.
