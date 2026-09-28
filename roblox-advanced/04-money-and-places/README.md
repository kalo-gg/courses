# 4. Money and places

*Purchases that are granted exactly once, passes and badges, and teleports that carry players and their data safely.*

Roblox Advanced · Medium

---

A purchase is money a player has already spent, so granting it twice costs you and granting it never costs them. This module covers developer products bound with `BindReceiptHandler` and made idempotent by the `PurchaseId` stored in the session-locked profile; game passes, badges and subscriptions, checked at the right moments; teleports with `TeleportAsync`, reserved servers and the two ways a teleport fails; and arriving with data, where the teleport data carries only an id and the real party waits in MemoryStore. It ends with a party that leaves the lobby and arrives in the arena intact, and nobody else getting in.

## Lessons

1. [Developer products](01-developer-products.md) — Bind a receipt handler per product, grant each PurchaseId once, and answer Processed only after the save lands.
2. [Game passes and badges](02-passes-and-badges.md) — Ask for pass ownership on join, grant from the purchase event, and award badges without letting one failed call stop the rest.
3. [Teleports](03-teleports.md) — Send a party to one reserved server with TeleportAsync, and catch both the call that throws and the teleport that fails later.
4. [Joining with data](04-teleport-data.md) — Send only a server-made id, keep the party in MemoryStore, release the lock before leaving, and check the claim on arrival.
