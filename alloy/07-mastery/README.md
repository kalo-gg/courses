# 7. The toolchain and mastery

*Migrate Luau with the lints, lay out a project, extend the toolchain, and hold the strict contracts.*

Alloy · Hard

---

The last module is about the tools around the language. It starts with migration: the lints that steer a Luau file toward Alloy, their groups and levels, and `alloy check`, `lint`, `flux` and `fmt`, with the directives and `@allow` for the cases where a lint is wrong. Then it lays out a project with the `[project]` and `[mount]` tables and the files `alloy build` derives from them. It covers the docs in the terminal, `alloy mcp` for coding agents, the language server, and ingots, the native extensions a project installs from a path or a GitHub release. It ends with the contracts Alloy holds at compile time, and with Luau strict mode on top of them. The capstone ships the satchel.

## Lessons

1. [From Luau to Alloy](01-from-luau.md) — Rename a Luau file to `.aly` and let the lints show you, one warning at a time, what to rewrite.
2. [Project layout](02-project-layout.md) — One `[mount]` table says where each folder lands, and `alloy build` writes the Rojo project, the sourcemap and the aliases from it.
3. [Docs, agents and ingots](03-extending-alloy.md) — Read the docs from the terminal, hand them to a coding agent, and add native extensions to the compiler.
4. [Strict by default](04-strict-by-default.md) — The contracts Alloy holds at compile time, Luau strict mode on top of them, and the one directive for an error you mean.
