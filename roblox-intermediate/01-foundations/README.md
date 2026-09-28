# 1. Client and server

*One server, one client per player, what each of them can see, and why the client must wait.*

Roblox Intermediate · Easy

---

A Roblox game runs on one server and on one client per player, and each of them holds its own copy of the world. This module covers how the work splits between them and why the server has the final say; what replicates, which is everything the server changes and almost nothing a client changes; where client code runs, starting from the recommended client entry point and `Players.LocalPlayer`; and why the client waits, because replication arrives in no guaranteed order and streaming may hold a distant part back. It ends with a client script that waits for what it needs and carries on without what it can skip.

## Lessons

1. [One game, many machines](01-one-game-many-machines.md) — One server holds the real game, each player's device runs a client with its own copy, and every message between them takes time.
2. [What replicates](02-what-replicates.md) — The server's changes reach every client. A client's changes stay on that device, apart from a few the engine accepts on purpose.
3. [Code on the client](03-code-on-the-client.md) — The client entry point, where client code can and cannot run, the four RunContext values, and Players.LocalPlayer.
4. [Waiting on the client](04-waiting-on-the-client.md) — The order a client starts in, why it waits for what arrives later, and what streaming holds back.
