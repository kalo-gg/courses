# 4. Wally in a Real Project

*Wiring the tree into a place with Rojo, getting your types back, and reading the errors you will meet.*

Wally · Medium

---

A manifest that resolves is not the same as a project that works. This module wires the installed tree into a place with Rojo, gets exported Luau types back after they are stripped by the link modules, keeps a project up to date without the command that would have done it, and reads the errors you will actually meet.

## Lessons

1. [Wiring it into a place](01-rojo-wiring.md) — Rojo maps the three installed directories into a place, and a package is a project file of its own.
2. [Getting your types back](02-types.md) — Link modules are bare require thunks, so exported types vanish until you put them back.
3. [Updating without wally update](03-updating.md) — The update command exists in the docs and panics in the release, so here is what to do instead.
4. [Reading the errors](04-troubleshooting.md) — Four errors you will actually meet, what each one really means, and the one that lies about failing.
