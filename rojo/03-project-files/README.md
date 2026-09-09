# 3. Project Files in Depth

*Property values, composing trees from paths and pure nodes, deciding what Rojo owns, and modern script settings.*

Rojo · Medium

---

Reading a project file read one; this module writes serious ones. You will set typed property values and attributes from JSON, compose a tree from several folders, nested projects and pure nodes, tell Rojo which instances it owns and which files to skip, and switch on modern script emission and custom sync rules — then write the arena's first project file as a project.

## Lessons

1. [Properties and attributes](01-properties-and-attributes.md) — Implicit values Rojo can type by itself, explicit { "Type": value } wrappers, and $attributes.
2. [Composing trees](02-composing-trees.md) — Mixing $path with pure children, $className on a Folder path, optional paths, and nested project files.
3. [What Rojo owns](03-what-rojo-owns.md) — $ignoreUnknownInstances, partially versus fully managed places, and globIgnorePaths.
4. [Modern scripts and sync rules](04-modern-scripts-and-sync-rules.md) — emitLegacyScripts: false for RunContext scripts, custom syncRules, and JSON with comments.
