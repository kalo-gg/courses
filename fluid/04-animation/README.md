# 4. Animation & Timing

*Springs that hand you the controls, lerps over a duration, and work on a clock.*

Fluid · Medium

---

Fluid's animation returns a pair — the value and a way to interfere with it — which is the shape to remember. This module covers springs and their control table, `lerp` and the two types it quietly cannot interpolate, `interval` for repeating work measured in hertz, and the scheduler step you can take over entirely, deferred phase and all.

## Lessons

1. [Springs](01-springs.md) — Two return values: where the value is now, and a control function for pushing it around.
2. [Lerping to a goal](02-lerp.md) — Move to a goal over a fixed duration, once you have checked the type can be interpolated.
3. [Work on a clock](03-interval.md) — Repeating work at a rate in hertz, with one free call before the clock starts counting.
4. [One step, three phases](04-the-scheduler.md) — A single step runs the intervals, flushes the deferred nodes, then advances the animators.
