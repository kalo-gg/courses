# 5. The Loop and CI

*Fast enough to keep running locally, honest enough to gate a pull request.*

Lest · Medium

---

A suite is only worth having if you run it. This module covers the machinery that makes that true on both ends: watch mode and --changed, which run only the specs a change can affect; snapshots, which record what a value looked like and make you review it; coverage, measured and gated on the native backend; and the reporters and exit-code contract that make a red build mean something specific.

## Lessons

1. [The save-run loop](01-the-loop.md) — Watch mode follows the require graph rather than a glob, and --changed does the same from a git ref.
2. [Snapshots](02-snapshots.md) — Record what a value serialized to, review the file, and know when a stored key is stale.
3. [Coverage](03-coverage.md) — Measure line coverage on the native backend, get a clean lcov file out, and gate a run on a minimum.
4. [Continuous integration](04-ci.md) — The exit-code contract, two invocations, a workflow file, and the reporters that feed a pull request.
