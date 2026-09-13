# Scribe

*Persistent player data for Roblox*

---

Scribe is a Luau library for player data built on ProfileStore: you declare the shape of a profile once, and it saves the data, holds the session lock so two servers cannot fight over it, replicates a copy to the client and hands you a typed accessor for every field. This course teaches the 2.3 release: declarators and bounds, containers, derived fields, who sees which field and how the client reads it, commands, the session lifecycle and saving, transactions, timers and cooldowns, offline profiles and migrations, purchases, leaderboards, testing and diagnostics. Three projects build one thing, the data layer of a small RPG called Emberfall: its inventory service, then its game server, then the whole thing with a client and an admin path. Every lesson runs against the real Scribe 2.3.0 sources in your browser, in the same Mock mode you would use in Studio.

- **Library** [Scribe](https://github.com/ericplane/Scribe)
- **Written by** ericplane
- **Verified against** 2.3.0
- **Course** 7 modules · 38 lessons on kalo.gg, 28 of them written out here
- **Authors** @crota

## Modules

| # | Module | Difficulty | Lessons |
| --- | --- | --- | --- |
| 1 | [Scribe Foundations](01-foundations/) | Easy | 4 |
| 2 | [Fields & Values](02-values/) | Easy | 4 |
| 3 | [Containers & Big Numbers](03-containers/) | Medium | 4 |
| 4 | [Replication & Commands](04-replication/) | Medium | 4 |
| 5 | [Sessions & Saving](05-sessions/) | Medium | 4 |
| 6 | [Production Data](06-production/) | Hard | 4 |
| 7 | [Shipping Emberfall](07-mastery/) | Hard | 4 |

The [API reference](reference.md) lists what the library exposes, verified against the version above.
