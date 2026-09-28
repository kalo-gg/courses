# 1. DataStores

*Keys, reads and writes, UpdateAsync, the limits, and when to save.*

Roblox Advanced · Easy

---

A DataStore keeps a player's data after the server that held it is gone, and every call to one is a web request that can fail. This module covers keys built from `player.User.Id`, `GetAsync` and `SetAsync` behind `pcall`, `UpdateAsync` for a key that other servers also write, the request budgets and size limits, retries that back off, and the three moments to save: when a player leaves, on an autosave loop, and in `BindToClose`. It ends with a server that saves every loaded player before it closes.

## Lessons

1. [DataStores](01-data-stores.md) — A value under a key that outlives every server, read and written by web requests that can fail.
2. [UpdateAsync](02-update-async.md) — Read, change and write a key in one call, which runs again when another server wrote first.
3. [Limits and retries](03-limits-and-retries.md) — What one server may spend, what one key may hold, and a retry that backs off instead of hammering a service that said no.
4. [When to save](04-when-to-save.md) — Save when a player leaves, on a timer, and when the server closes, and wait for every save before it does.
