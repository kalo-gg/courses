# Luau Advanced

*Performance, metaprogramming and the runtime underneath*

---

Luau is fast, and most advice about making it faster is folklore. This course starts with what the compiler and VM actually do, then covers the parts of the language you reach for when performance or expressiveness is the point: the type system past the basics, buffers and vectors for packed data, coroutines for work that pauses, and the full metamethod table. It ends where the language does, with the limits and the habits that respect them. Two projects build one thing, a particle simulation: its packed buffer after the third module, then the whole stepped simulation at the end.

- **Library** [Luau](https://github.com/luau-lang/luau)
- **Written by** Roblox
- **Verified against** 0.733
- **Course** 6 modules · 32 lessons on kalo.gg, 24 of them written out here
- **Authors** @crota

## Modules

| # | Module | Difficulty | Lessons |
| --- | --- | --- | --- |
| 1 | [How Luau Runs](01-foundations/) | Easy | 4 |
| 2 | [The Type System, Deeply](02-types/) | Medium | 4 |
| 3 | [Buffers & Vectors](03-data/) | Medium | 4 |
| 4 | [Coroutines](04-concurrency/) | Medium | 4 |
| 5 | [Metamethods in Full](05-metaprogramming/) | Hard | 4 |
| 6 | [Shipping Fast Luau](06-mastery/) | Hard | 4 |

The [API reference](reference.md) lists what the library exposes, verified against the version above.
