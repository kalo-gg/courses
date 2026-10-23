# 1. Larvae Foundations

*What larvae replaces, how to install it beside luau-lsp, and the one file every command reads.*

Larvae · Easy

---

Most Luau projects run three or four tools that each have their own config and their own idea of what a project is. larvae is one binary that formats, lints, rewrites requires, bundles and serves an editor from one larvae.toml, and it leaves the types to luau-lsp. It covers what larvae does, how to install it, how larvae.toml is layered, and what the first hour looks like command by command, ending with the first config you write.

## Lessons

1. [What is larvae?](01-what-is-larvae.md) — A Rust toolchain that owns the whole build, instead of four tools that each own a slice of it.
2. [Installing larvae](02-installing-larvae.md) — One binary by any route, luau-lsp beside it for the types, and three switches that adopt only part of it.
3. [larvae.toml](03-the-config.md) — One file, layered, with short forms for the three keys every project sets.
4. [The first hour](04-the-first-hour.md) — What larvae init writes for your project, the six commands of the first hour, and the Rojo handoff.
