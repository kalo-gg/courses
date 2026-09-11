# 6. Modules & Mistakes

*Split a program across files, survive the errors it will throw, and write it so someone can read it.*

Luau Fundamentals · Hard

---

A program that fits in one file eventually stops fitting. This module splits one across files with require, then spends the rest of its time on what goes wrong: the error that a missing value throws one line too late, the difference between crashing and handling, and the habits that let the next person read what you wrote. It ends with the whole text adventure.

## Lessons

1. [Modules and require](01-modules.md) — A file that returns a table becomes a module, and require gives every caller the same copy.
2. [Writing readable Luau](02-readable-luau.md) — Naming, locals over globals, small functions, and comments that say why rather than what.
3. [When it goes wrong on purpose](03-handling-errors.md) — error stops the script, pcall catches it, and assert is the short way to demand a condition.
4. [Guarding against nil](04-nil-safety.md) — Most crashes are a nil used one line after it appeared, and four small habits prevent nearly all.
