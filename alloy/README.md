# Alloy

*Luau, with the parts it is missing*

---

Every Luau file is already an Alloy file. Alloy adds what Roblox code keeps writing by hand: safe access, structs, payload enums with exhaustive `match`, traits, Results and async, typed remotes, attributes and macros. It compiles each construct to fixed Luau on the same line, so a stack trace still points at the line you wrote. This course starts with the operators and sugar, then covers structs, enums and pattern matching, interfaces, traits and extensions, modules, Results and async, and the compile-time features: attributes, macros, markup and inline tests. Then it covers the Roblox features (sides, typed remotes and wire types, parallel Luau), and ends with the toolchain and the strict-by-default contracts. It targets Alloy 0.1.0. Across two projects you build a satchel: first the inventory module, then the whole system on Roblox, with saves, remotes and tests.

- **Library** [Alloy](https://github.com/alloy-luau/alloy)
- **Written by** sav
- **Verified against** 0.1.0
- **Course** 7 modules · 37 lessons on kalo.gg, 28 of them written out here
- **Authors** @crota · endorsed by @sav

## Modules

| # | Module | Difficulty | Lessons |
| --- | --- | --- | --- |
| 1 | [Foundations](01-foundations/) | Easy | 4 |
| 2 | [Structs and enums](02-structs-and-enums/) | Easy | 4 |
| 3 | [Types, traits and extensions](03-types-and-traits/) | Medium | 4 |
| 4 | [Modules, Results and async](04-modules-and-async/) | Medium | 4 |
| 5 | [Compile-time Alloy](05-compile-time/) | Medium | 4 |
| 6 | [Alloy on Roblox](06-roblox/) | Hard | 4 |
| 7 | [The toolchain and mastery](07-mastery/) | Hard | 4 |

The [API reference](reference.md) lists what the library exposes, verified against the version above.
