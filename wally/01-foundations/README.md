# 1. Wally Foundations

*What a package manager buys you, how to install the CLI, and the registry sitting behind it.*

Wally · Easy

---

Every Roblox project eventually copies a module out of someone else's place file and loses track of which version it was. Wally replaces that with a manifest: you name what you depend on, it fetches exactly those versions, and anyone who clones the repository gets the same tree. This module installs the tool, writes your first manifest, and explains what the registry behind it actually is.

## Lessons

1. [What is Wally?](01-what-is-wally.md) — A package manager for Roblox: a manifest names your dependencies and a registry serves them.
2. [Installing Wally](02-installing-wally.md) — Rokit installs the pinned CLI, and the version you get is the one this course is written against.
3. [Your first manifest](03-first-manifest.md) — wally init writes four required keys, and every one of them means something you should choose.
4. [The registry behind it](04-the-registry.md) — One git repository holds every published manifest, and knowing its shape explains a lot.
