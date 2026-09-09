# 1. Lest Foundations

*One binary that carries the framework inside it — and refuses to fake an environment.*

Lest · Easy

---

Lest is a testing toolchain rather than a testing library: the framework is compiled into the CLI, so the runner and the framework can never be different versions. This module covers what that buys you and the position it takes on mocking the engine, then gets you from nothing to a passing spec: installing the binary, writing the lest.toml that names your suites, and reading the report a first run prints — and a failing one.

## Lessons

1. [What is Lest?](01-what-is-lest.md) — A toolchain rather than a library, a position on mocking, and six places a test can run.
2. [Installing and scaffolding](02-installing-lest.md) — One binary, then lest init writes a config, an example spec and an alias, and what not to commit.
3. [Suites and lest.toml](03-configuring-suites.md) — Zero config runs a suite named specs; lest.toml names your own, picks a backend and sets the defaults.
4. [Your first spec](04-first-spec.md) — Write a spec, run it, read a passing report, and then read a failing one.
