# 1. Scribe Foundations

*What Scribe does for you, how to install 2.3, and a first profile that saves, loads and survives a rejoin.*

Scribe · Easy

---

Scribe is a Luau library for player data: you write the shape of a profile once and it saves the data, holds the session lock, replicates a copy to the client and gives you a typed accessor for every field. This module covers the problem it removes, installs the 2.3 release as the one shared module both realms require, reads the Emberfall template line by line, and ends with your first session: a player joins, their data loads, a write lands, they leave, and the number is still there when they come back.

## Lessons

1. [What is Scribe?](01-what-is-scribe.md) — Write the shape of your data once and Scribe saves it, locks it, replicates it and types it.
2. [Installing Scribe 2.3](02-installing-scribe.md) — One Wally line, one shared module both realms require, and the Studio setting that decides whether anything saves.
3. [The Emberfall template](03-the-emberfall-template.md) — Read the docs' example template line by line: plain values, declarators, nested tables and the three options every bundle needs.
4. [Your first session](04-your-first-session.md) — A player joins, the profile loads, a write lands, they leave, and the number is still there when they come back.
