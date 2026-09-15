# 1. Blink Foundations

*What an IDL buys you over raw remotes, how to install the compiler, and your first generated module.*

Blink · Easy

---

A Roblox RemoteEvent sends whatever table you hand it, serialised by Roblox, at whatever size that turns out to be. Blink asks you to describe the traffic instead: a small file naming each event, who fires it, how reliable it is, and exactly what data rides along. From that it compiles a server module and a client module that pack every field into bytes. This module installs the compiler, writes your first description, and reads what came out.

## Lessons

1. [What is Blink?](01-what-is-blink.md) — An IDL compiler: you describe your network traffic, it writes the buffer code for you.
2. [Installing Blink](02-installing-blink.md) — Rokit pins the compiler per project, and the version you get is the one this course uses.
3. [Your first description](03-first-definition.md) — Two output options, one event, and the compiler writes both sides for you.
4. [Reading what it generated](04-reading-the-output.md) — One frozen table per side, and the method names tell you which side you are on.
