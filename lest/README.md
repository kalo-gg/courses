# Lest

*Testing for Luau*

---

Lest is a single binary carrying the test framework inside it, so the runner and the framework can never disagree. This course covers writing specs with describe, it and expect, every matcher and where Luau's semantics differ from Jest's, the six backends and the no-emulation position behind them, and the watch, snapshot, coverage and CI machinery that makes a suite worth keeping. Two projects build the spec suite for a shop's cart and checkout — the modules every sample tests — from the first describe to a gated CI run.

- **Library** [lest](https://github.com/lest-luau/lest)
- **Verified against** 0.6.0
- **Course** 6 modules · 32 lessons on kalo.gg, 24 of them written out here
- **Authors** @crota · endorsed by @crota

## Modules

| # | Module | Difficulty | Lessons |
| --- | --- | --- | --- |
| 1 | [Lest Foundations](01-foundations/) | Easy | 4 |
| 2 | [Writing Tests](02-writing-tests/) | Easy | 4 |
| 3 | [Matchers](03-matchers/) | Medium | 4 |
| 4 | [Backends](04-backends/) | Medium | 4 |
| 5 | [The Loop and CI](05-loop-and-ci/) | Medium | 4 |
| 6 | [Suite Design & Capstone](06-mastery/) | Hard | 4 |

The [API reference](reference.md) lists what the library exposes, verified against the version above.
