# 5. Sessions & Saving

*From PlayerAdded to the last save: waiting for data, hooks, flushes, transactions, and timers that outlive a session.*

Scribe · Medium

---

A profile does not exist the instant a player joins, and it is not on disk the instant you write it. This module follows one session end to end: WaitForData and its seven reasons, OnPlayerInit and OnPlayerLeaving, the autosave, Flush and what its false really means, Batch against Transaction and the rollback only one of them gives you, then Timed fields and cooldowns, two answers to two different questions about time. It closes with the second project: Emberfall's server, with a shop, a daily reward and a leaving hook.

## Lessons

1. [The session lifecycle](01-lifecycle.md) — WaitForData and its seven reasons, the hook that runs before Ready, the hook that runs before the last save, and the signals.
2. [Saving](02-saving.md) — The autosave, the save on leave, Flush and what its false really means, and the two calls that watch save state.
3. [Batch and Transaction](03-transactions.md) — Batch coalesces, Transaction rolls back, Flush makes it durable; and why one player never needs a protocol.
4. [Timers and cooldowns](04-timers-and-cooldowns.md) — A Timed field is a value that expires; a cooldown is a gate with no value. Picking the wrong one is the common mistake.
