# 1. Foundations

*What the engine is, the services that hold a place together, where scripts run, and your first part.*

Roblox Fundamentals · Easy

---

Every Roblox game is one tree of instances rooted at `game`, and every script changes that tree. This module shows you the tree, how Studio edits and tests it, and where a script's `print` ends up. It then covers the services at the top of the tree and fetching them with `GetService`, the three kinds of script and the layout Roblox recommends (one server Script in ServerScriptService, with ModuleScripts for the rest), and ends with your first part: an anchored platform built in code, with its `Parent` set last.

## Lessons

1. [What the engine is](01-what-is-the-engine.md) — A place is one tree of instances rooted at game. Scripts change the tree, and the engine does everything else.
2. [Services](02-services.md) — game:GetService finds a service by its class, and the container you choose decides who can see what is inside it.
3. [Scripts and where they run](03-scripts-and-where-they-run.md) — Script, LocalScript and ModuleScript, RunContext, and the recommended layout: one entry point for each side and modules for the rest.
4. [Your first part](04-your-first-part.md) — Instance.new, four properties, and Parent set last: an anchored platform in the world.
