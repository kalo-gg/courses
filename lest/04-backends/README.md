# 4. Backends

*The same suite, run somewhere it can actually be true — and a real place for the tests that need one.*

Lest · Medium

---

A suite's backend is where its specs execute. You choose it once, and everything downstream — reporters, snapshots, filters, CI output — neither knows nor cares. This module covers what each backend gives up and what it buys, the two properties that genuinely differ between them, how to keep a slow engine suite out of your local loop without letting it rot, and the ways the three real engines are not interchangeable.

## Lessons

1. [Six places a test can run](01-the-six.md) — One embedded VM, two spawned runtimes, and three real engines, and how to pick the cheapest one that is true.
2. [Async and isolation](02-async-and-isolation.md) — Two properties that genuinely differ between backends, and neither is a detail.
3. [Engine suites](03-engine-suites.md) — A real place, an API key that never touches your config, and a suite that stays out of the local loop.
4. [The three engines](04-cloud-studio-gargantuan.md) — One suite, two Roblox contexts that are not the same, and the experimental third engine.
