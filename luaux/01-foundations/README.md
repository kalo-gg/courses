# 1. LuauX Foundations

*Write JSX for Roblox, ship ordinary Luau, carry no runtime.*

LuauX · Easy

---

LuauX is a compiler, not a library. A `.luaux` file holds JSX-style markup; `luaux build` turns it into a `.luau` file that calls whichever UI library you already use, and nothing is added to the game. This module says what that buys you, installs the compiler, writes the build config that points Rojo at the output, and compiles your first file into a mounted screen.

## Lessons

1. [What is LuauX?](01-what-is-luaux.md) — Markup in, plain Luau out, nothing left behind: what LuauX is and what it leaves to your library.
2. [Why LuauX?](02-why-luaux.md) — The three things markup buys you, the two things it refuses to do, and your first conversion.
3. [Installing and building](03-installing-luaux.md) — Install the binary, scaffold a config, compile src into build, and point Rojo at the output.
4. [Your first file](04-first-file.md) — Write a .luaux file, read the .luau it becomes, and mount it from a LocalScript next to it.
