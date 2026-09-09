# 5. Actions, Springs & Context

*Run code on instances with actions, turn property changes into sources, animate with springs, and share values with context.*

Vide · Medium

---

Property tables say what an instance looks like; sometimes you need to *do* something with it: read a property back, connect to a changed signal, or hand it elsewhere. Actions are the hook for that. This module also covers the two helpers that make Vide UI feel alive and stay organized: springs, which animate any source towards its target, and context, which passes values down a tree of components without threading props through every layer.

## Lessons

1. [Actions](01-actions.md) — Attach a callback to a property table; it runs with the finished instance.
2. [Listening with changed](02-changed.md) — Turn a property the engine changes into a source with the changed action, and bind both ways.
3. [Context](03-context.md) — Provide a value to a subtree of components without passing it through every props table.
4. [Springs](04-springs.md) — Animate any source with spring physics; tune period and damping; nudge with the control.
