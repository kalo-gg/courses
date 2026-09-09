# 5. Worms & Extensions

*Teaching larvae a language it has never seen — without giving it a second formatter.*

Larvae · Hard

---

A worm is an extension that claims a file extension larvae does not understand and answers for it. The design is the interesting part: a worm compiles, formats and lints, but hands back less than you expect for the last two, so a claimed file still formats and reports like Luau. What a worm is, where one comes from, how a project configures what it adds, and how to write one — taught through luaux-worm, the tooling behind Kalo's LuauX course.

## Lessons

1. [What a worm is](01-what-is-a-worm.md) — A guest larvae hands a file to — in an embedded VM, in a wasm interpreter, or as a process of its own.
2. [Installing a worm](02-installing-a-worm.md) — Where a worm comes from — a release, a crate, or a directory on disk — and the two commands that fetch it.
3. [Configuring a worm](03-configuring-a-worm.md) — Everything a worm adds lives under its key: lints, formatter options, settings, and its share of larvae's rules.
4. [Writing one](04-writing-a-worm.md) — worm.toml declares what the worm claims, reports and offers, and two commands try it before it is installed.
