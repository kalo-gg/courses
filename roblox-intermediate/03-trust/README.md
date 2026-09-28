# 3. Trust

*Assume every client is hostile, check every argument, limit every request, and let the server decide.*

Roblox Intermediate · Medium

---

An exploiter controls their own client completely, so everything a client sends is a claim, not a fact. This module covers what an exploiter can and cannot do; validating each argument's type, value and instance with `typeof`, `math.isfinite`, `utf8.len`, `IsA` and `IsDescendantOf`; per-player rate limits, from a cooldown to a token bucket; and server authority, where the client sends intent and the server works out every price and reward from its own tables. It ends with the arena's first project: a loadout API the server validates.

## Lessons

1. [Never trust the client](01-never-trust-the-client.md) — An exploiter controls their own client completely, and nothing else. Know where that line runs before you write a handler.
2. [Validating arguments](02-validating-arguments.md) — Check each argument's type, then its value, then which instance it is and whose, before the handler uses it.
3. [Rate limits](03-rate-limits.md) — A valid request can still arrive too often. Limit each player with a cooldown or a token bucket, drop the excess, and forget them when they leave.
4. [Server authority](04-server-authority.md) — The client says what the player wants to do. The server works out what it costs and earns from its own tables, applies it, then tells the clients.
