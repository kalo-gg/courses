# 6. Roblox Integration

*Instances, players and input as data, and component replication with nothing but JECS and RemoteEvents.*

JECS · Medium

---

JECS knows nothing about Roblox, which is exactly why it is easy to fit in. This module uses only the core library and Roblox services: keep the Instance in its own component and let hooks own its lifetime, give every player an entity and turn input into data, then replicate component data with the official networking pattern, server side and client side. It closes with the second project, which binds the world to Instances and players.

## Lessons

1. [Instances and entities](01-instances-and-entities.md) — Keep the Instance in its own component, let hooks own its lifetime, and find the entity behind a model.
2. [Players and input as data](02-players-and-input.md) — Give every player an entity, spawn their character into it, and turn input into components a system consumes.
3. [Replicating components](03-replicating-components.md) — Match component ids across realms, tag what replicates, record changes with signals and flush a per-frame diff.
4. [Mirroring on the client](04-mirroring-on-the-client.md) — Map server ids to local entities, apply set and removed, and learn why the client never adopts a server id.
