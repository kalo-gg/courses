# 6. Production Data

*Offline profiles, migrations that fail closed, Robux and soft-currency purchases, and leaderboards from one line of config.*

Scribe · Hard

---

Everything so far happened inside one live session. A shipped game also has to act on profiles nobody is playing: a support ticket, a GDPR request, a template that changed shape since last month. This module covers the offline API and the session lock that makes every offline write fail closed, the numbered migration chain and the one guard that never fires, products and passes with the receipt path Roblox retries until the save confirms, soft-currency purchases that are all or nothing, and leaderboards that read from a cache and never let a player touch the store.

## Lessons

1. [Offline profiles](01-offline-profiles.md) — Read, edit, export and erase a stored profile for a player who is not here, and why every write fails closed.
2. [Migrations](02-migrations.md) — Evolve your own template with a numbered chain that fails closed, and key each step off something reconcile cannot fake.
3. [Products, passes and purchases](03-monetization.md) — Declare products and passes next to the template, check what a player owns on the server, and spend soft currency as one write.
4. [Leaderboards](04-leaderboards.md) — Name a field, and Scribe keeps every player's score in an OrderedDataStore and hands you the top hundred from a cache.
