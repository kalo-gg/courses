# 6. Alloy on Roblox

*One codebase for both sides, typed remotes with a wire layout, and parallel Luau the compiler guards.*

Alloy · Hard

---

A Roblox game is two programs that share code: the server and every client. This module covers the features Alloy adds for that split. It starts with sides: `.server.aly` and `.client.aly` files, and `@cfg`, which keeps one shared startup honest about where each half runs. Then typed remotes: one `remote` declaration gives both sides a checked channel, with rate limits, validation and timeouts as attributes. Then the wire layout, where widths such as `@u8` pack a payload into a buffer and the receiver drops any packet that does not fit. It ends with parallel Luau: `parallel do`, the statements the compiler refuses inside it, and typed actor messages. Roblox Fundamentals is assumed. Roblox Intermediate's remotes lesson is recommended but not required.

## Lessons

1. [Sides and @cfg](01-sides-and-cfg.md) — Say which side a script runs on with its file name, and which side a function, statement or local belongs to with @cfg.
2. [Typed remotes](02-remotes.md) — Declare a remote once, fire and handle it from both sides, and let attributes limit, validate and time out what crosses.
3. [Wire types](03-wire-types.md) — How a remote's payload travels: widths, buffers and tables, and the checks every packet passes on arrival.
4. [Parallel Luau](04-parallel-and-actors.md) — Run a block in the parallel phase, let the compiler refuse what the engine would, and talk to actors with typed messages.
