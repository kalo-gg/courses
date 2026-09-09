# 1. Rojo Foundations

*Why files beat Studio-only editing, how to install Rojo, and how a project file turns folders into a game.*

Rojo · Easy

---

Roblox Studio stores your whole game in one binary place file, which means no real version control, no editor of your choice and no automation. Rojo puts the source of truth on disk instead: a project file says where each folder lands in the game, and a Studio plugin keeps the place in sync while you edit. This module installs the tools, creates a first project and reads its project file line by line.

## Lessons

1. [What is Rojo?](01-what-is-rojo.md) — A CLI and a Studio plugin that keep a folder of files and a Roblox place in sync.
2. [Installing Rojo](02-installing-rojo.md) — Pin the CLI with Rokit, install the matching Studio plugin, and check the versions line up.
3. [Your first project](03-first-project.md) — rojo init, build a place, serve it, connect the plugin, and watch a file change land in Studio.
4. [Reading a project file](04-reading-a-project-file.md) — The keys of tree, service inference, $path, and the top-level settings around them.
