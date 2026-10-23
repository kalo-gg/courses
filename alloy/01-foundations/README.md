# 1. Foundations

*What Alloy is, a first build, the safe-access operators, and a first script in the recommended style.*

Alloy · Easy

---

Every Luau file is already an Alloy file. This module shows what Alloy adds and what it promises: each construct compiles to fixed Luau on the line you wrote it, so a stack trace still points at your source. It installs the compiler, runs `alloy init` and `alloy build`, and reads the `alloy.toml` the wizard writes. Then it covers the safe-access operators, `?.`, `??`, `!`, `->` and `=>`, and ends with a first script in the recommended style: `const`, `new` and the initializer, the ternary, array literals, destructuring, `in` and `is`.

## Lessons

1. [What is Alloy?](01-what-is-alloy.md) — A strict superset of Luau that compiles each construct to fixed Luau on the same line.
2. [Installing and building](02-installing-alloy.md) — Install the binaries, let `alloy init` write the project, and keep `alloy build -W` compiling.
3. [Safe access](03-safe-access.md) — Read through values that may be nil with `?.`, fall back with `??`, insist with `!`, and find children with `->` and `=>`.
4. [Your first Alloy script](04-first-script.md) — Build parts with `new` and an initializer, name things with `const`, and meet arrays, destructuring, `in` and `is`.
