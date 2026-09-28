# 7. Shipping a place

*A place organised into modules, debugged in Studio, cheap to run with many parts, and clean enough to run for days.*

Roblox Fundamentals · Hard

---

This module turns scripts that work into a place you can ship. It splits one long Script into an entry point and ModuleScripts, puts each in the container the docs recommend, and keeps requires from going in a circle. It covers debugging in Studio: reading an error to its line, breakpoints and the Watch window, and the Test, Run and Server & Clients modes. Then it looks at the cost of per-part code every frame, replacing a thousand connections with one, and handing continuous motion to constraints. It ends with cleanup: what leaks on a server that runs for days, and the habits that hold whatever the engine cleans up for you. The capstone ships the whole obby.

## Lessons

1. [Organising a place](01-organising-a-place.md) — One entry Script, ModuleScripts for everything else, each container used for its job, and requires that never go in a circle.
2. [Debugging in Studio](02-debugging-in-studio.md) — Read an error to its line, pause a script with breakpoints and Watch, and pick the test mode that shows the bug.
3. [One loop, many parts](03-one-loop-many-parts.md) — Drive every tagged part from one connection over a table, and hand continuous motion to the engine.
4. [Cleaning up](04-cleaning-up.md) — What leaks on a server that runs for days, and the disconnect-and-clear habits that hold under every setting.
