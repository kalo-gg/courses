# 4. Builds & Profiles

*The daily loop: process, watch, transform, fold constants, and ship a debug and a release build from one config.*

Larvae · Medium

---

larvae process can do more than rewrite requires. This module is the build loop as a project lives with it: what [process] reads and writes and how the derived Rojo project keeps Studio in sync, the rules that transform code on the way out, the defines that fold constants into it, the profiles that give you a debug and a release build from one file, and the minify and bundle steps that shrink what ships.

## Lessons

1. [The build loop](01-the-build-loop.md) — What larvae process reads and writes, the generator that keeps line numbers, and the Rojo project it derives.
2. [Rules](02-rules.md) — Every transformation is off until you name it, and three of them share one footgun.
3. [Defines and profiles](03-defines-and-profiles.md) — Compile-time constants that fold away, and one config that builds debug and release.
4. [Minify and bundle](04-minify-and-bundle.md) — Shrink the release build with the dense generator, and collapse the project into one file that loads on demand.
