# 4. Blink in a Real Project

*Output paths, casing, scopes, imports, and how the two generated modules reach your game code.*

Blink · Medium

---

A description that compiles is not a network layer. This module covers the options that decide where the output goes and what it is called, splitting a growing description across files with scopes and imports, wiring the two generated modules into a place so both sides can require them, and reading the errors you will actually meet — including the one the compiler does not give you.

## Lessons

1. [Options](01-output-options.md) — Thirteen options, and the error message that lists them is the only complete reference.
2. [Scopes and imports](02-scopes-and-imports.md) — Group definitions with scope, split files with import, and never import the same file twice.
3. [Getting the modules into your game](03-wiring-it-in.md) — Two generated modules, two realms, and one folder that both sides can actually see.
4. [Reading the errors](04-reading-errors.md) — Eight error codes, one exit code, and the exit code is always zero.
