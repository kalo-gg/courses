# 4. Coroutines

*Functions that pause and resume, the values that pass through, and the scheduler you build from them.*

Luau Advanced · Medium

---

A coroutine is a function that can stop in the middle and be started again later, with values passing in both directions. It is the mechanism under every wait, every task scheduler and every generator you have used. This module builds one from scratch, covers the error handling that surrounds it, and finishes by writing a small scheduler.

## Lessons

1. [A function that pauses](01-coroutine-basics.md) — create, resume, yield and status: four calls that turn a function into something you can stop halfway.
2. [Values in both directions](02-passing-values.md) — yield is a call that returns when someone resumes you, and that one sentence explains the plumbing.
3. [wrap, and what happens to errors](03-wrap-and-errors.md) — wrap hides the thread behind a plain function and lets errors through; create keeps both in your hands.
4. [Writing a scheduler](04-a-scheduler.md) — A queue of coroutines resumed in turn is the whole idea behind every task library you have used.
