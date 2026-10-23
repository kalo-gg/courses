# 5. Tooling & Adoption

*Install it, wire it into Rojo, and let the compiler catch what Studio used to.*

LuauX · Medium

---

LuauX earns its place through the boring parts: a build step beside `rojo serve`, errors before a sync rather than after, and editor support split between the larvae toolchain and LuauX's own extension. The CLI and its five commands, what the compiler checks and the lint it ships, the `luaux` worm that gives `.luaux` files a formatter and a linter, the extension that gives them completions and hover, and adopting it one file at a time with a check in CI.

## Lessons

1. [The CLI](01-cli.md) — Five commands and where each belongs: the terminal, the watch loop beside Rojo, or CI.
2. [What the compiler catches](02-diagnostics.md) — Tags checked against 356 classes, attributes against 899, a lint with three levels, and what is still yours to avoid.
3. [In your editor](03-editor.md) — The larvae worm that lints and formats .luaux files, the LuauX extension that types them, and what each configures where.
4. [One file at a time](04-adoption.md) — Convert a project without a big bang: in place first, scan for candidates, a check in CI, a build directory last.
