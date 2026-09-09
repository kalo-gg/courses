# 2. Rewriting Requires

*Write one require, emit whichever shape the runtime needs — and be told when it could not work.*

Larvae · Easy

---

The part most projects adopt larvae for. You write one alias once; larvae emits a Roblox string, a filesystem path or an instance chain depending on the target, refuses a require that crosses a realm the runtime will not honor, and lets one corner of the project build differently through overrides. The module closes on larvae check — the whole-graph gate that CI runs and no level can silence.

## Lessons

1. [Aliases and targets](01-aliases-and-targets.md) — One source spelling, three possible outputs, and the indexing style that decides whether a chain waits.
2. [Realms](02-realms.md) — The check a formatter cannot do for you: which containers reach which, and the setting for clones.
3. [Overrides and mounts](03-overrides.md) — When one corner of the project has to build differently, and where larvae learns what lands where.
4. [The require gate](04-the-require-gate.md) — larvae check asks the questions only a whole require graph can answer, and two of them no level can silence.
