# 6. Structuring a Real Game

*Plan a repository, add places, libraries, tests and cross-references, then ship the arena with every rule in place.*

Rojo · Hard

---

Everything so far was a rule or a command. This module turns them into decisions: which layer — files, project file or settings — owns each instance, how one repository serves several places, how a library ships as a model with a test place beside it, and how instances reference each other with ids. The final project grows the arena skeleton from Project 1 into a shippable repository.

## Lessons

1. [Planning a repository](01-planning-a-repository.md) — Decide which layer owns each instance before you write a line: files, the project file, or the settings beside it.
2. [Several places, one repository](02-several-places.md) — One project file per place, shared code mapped once, and settings that stop at each project file's edge.
3. [Linking instances: $id and Ref properties](03-refs-and-ids.md) — Give an instance an id and point a Ref property at it from JSON, the way Rojo has done since 7.5.0.
4. [Libraries and test places](04-libraries-and-tests.md) — Ship a library as a model whose root is a ModuleScript, and build a test place beside it that keeps the specs in.
