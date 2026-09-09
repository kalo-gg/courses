# 3. Formatting & Linting

*Sixty-four lints, four levels, and a migration that is mostly a paste — once you know about two keys.*

Larvae · Medium

---

larvae aims at stylua and selene parity: both config files and both comment spellings keep working, and the lints keep their levels. What it leaves out is that larvae's defaults are not stylua's — three formatter options are its own opinion and on by default. Levels and groups, flag comments, tuning one lint, the formatter past stylua, and the two recommended keys that decide what your first run does, then Project 1: Lantern adopts larvae.

## Lessons

1. [Levels and groups](01-levels.md) — Four levels, six lints that deny by default, and groups that never wake a lint that is allowed on purpose.
2. [Flag comments](02-flag-comments.md) — Talking to the tool from inside the file, and why the vocabulary is deliberately small.
3. [Tuning the linter](03-tuning-the-linter.md) — Start from nothing with recommended = false, set options for one lint, and read a lint's own explanation.
4. [Formatting](04-formatting.md) — Every stylua option, and the three that are not stylua's, which are on by default.
