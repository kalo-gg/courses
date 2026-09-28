# 3. Across servers

*MessagingService for broadcasts, and MemoryStore for fast shared state.*

Roblox Advanced · Medium

---

Some state belongs to no single server: an announcement for everyone, the list of open servers, the line for the next round. This module covers MessagingService, which sends a message to every subscribed server on a best-effort basis; MemoryStore sorted maps, for ordered live data that expires on its own; queues, which hand each item to one reader and bring back what was never removed; and hash maps, the memory quotas, and which store fits which need. It ends with the course's first project: a session-locked profile store.

## Lessons

1. [MessagingService](01-messaging-service.md) — Publish a message on a topic, and every server subscribed to it receives it, usually within a second or two, but not always.
2. [MemoryStore sorted maps](02-sorted-maps.md) — Fast shared data that every server can read in order, and that expires on its own when nobody refreshes it.
3. [MemoryStore queues](03-queues.md) — One shared line: each read hides its items from other servers, and an item nobody removes comes back by itself.
4. [Hash maps, quotas and choosing a store](04-choosing-a-store.md) — The third MemoryStore structure, the quotas every structure shares, and which store fits which need.
