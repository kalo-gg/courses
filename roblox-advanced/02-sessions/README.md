# 2. Sessions

*One owner per player's data, schemas that change safely, leaderboards, and versions for rollback and erasure.*

Roblox Advanced · Medium

---

For a moment, two servers can hold the same player's data, and a DataStore will take a save from both. This module gives each player's data one owner: a session lock in the key's metadata, claimed with UpdateAsync, refreshed by every save, and taken over once it goes stale. Then it keeps old saves loading as your data changes shape, with a template to reconcile against and migrations that run in order. It puts scores in an OrderedDataStore for a global leaderboard, and ends with versions for rolling a player back, the user IDs and metadata every write must carry, and key names an erasure request can find.

## Lessons

1. [Session locking](01-session-locking.md) — One server owns a player's data at a time: a lock in the key's metadata, claimed with UpdateAsync, refreshed by every save, and taken over when it goes stale.
2. [Schemas and migrations](02-schemas-and-migrations.md) — Bring every old save up to date as it loads: a template for new fields, and numbered migration steps, in order, for changed ones.
3. [Leaderboards with OrderedDataStore](03-ordered-data-stores.md) — Integer scores in an OrderedDataStore, read back sorted a page at a time, and how often a server can afford to write and read them.
4. [Versions, metadata and erasure](04-versions-and-erasure.md) — Roll a key back to how it stood at a time, keep its user IDs and metadata on every write, and name keys so an erasure request finds them.
