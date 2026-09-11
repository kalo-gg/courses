# Larvae

*One toolchain for Luau*

---

larvae replaces the usual pile of stylua, selene, darklua and a language server with a single Rust toolchain. This course covers the config every command reads, the require rewriter and its realm check, formatting and linting with a migration path from the tools you already run, builds with rules and profiles, worms — the extension mechanism that lets larvae handle languages that are not Luau at all — and the editor server and CI gates a team runs it through. Two projects bring one repository, **Lantern**, under larvae: first its config and lint migration, then a release build with profiles, a worm for its markup and the workflow that gates it.

- **Library** [larvae](https://github.com/larvae-luau/larvae)
- **Verified against** 0.8.1
- **Course** 6 modules · 32 lessons on kalo.gg, 24 of them written out here
- **Authors** @crota, @sav · sponsored by @sav

## Modules

| # | Module | Difficulty | Lessons |
| --- | --- | --- | --- |
| 1 | [Larvae Foundations](01-foundations/) | Easy | 4 |
| 2 | [Rewriting Requires](02-requires/) | Easy | 4 |
| 3 | [Formatting & Linting](03-fmt-and-lint/) | Medium | 4 |
| 4 | [Builds & Profiles](04-builds/) | Medium | 4 |
| 5 | [Worms & Extensions](05-worms/) | Hard | 4 |
| 6 | [Larvae for a Team](06-mastery/) | Hard | 4 |

The [API reference](reference.md) lists what the library exposes, verified against the version above.
