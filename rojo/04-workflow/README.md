# 4. The Rojo Workflow

*Serve while you work, build for releases, feed your editor a sourcemap, lay out the repository, and read the log.*

Rojo · Medium

---

With the file rules in hand, the day-to-day loop is a handful of commands and the habits around them: serving while you work and building places for releases, giving your language server a sourcemap so instance paths resolve to files, a folder layout that tells you where every file runs, and the errors, warnings and silences Rojo produces when a project does not sync the way you expected.

## Lessons

1. [Serve and build](01-serve-and-build.md) — rojo serve options and project settings, rojo build output formats and --watch.
2. [Sourcemaps and editors](02-sourcemaps-and-editors.md) — rojo sourcemap tells your language server which file backs which instance; luau-lsp reads it for you.
3. [Layout conventions](03-layout-conventions.md) — A folder layout that keeps server, client, shared, GUI, assets and packages apart and maps cleanly.
4. [When sync goes wrong](04-debugging-sync.md) — Read Rojo's errors, warnings and silences, then fix a project that fails to build for five different reasons.
