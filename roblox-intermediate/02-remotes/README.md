# 2. Remotes

*RemoteEvents and RemoteFunctions, what survives the trip, and when to accept a dropped message.*

Roblox Intermediate · Easy

---

Remotes are the only way the client and the server call each other. This module creates a RemoteEvent on the server and fires it both ways, with the engine supplying the player a client cannot forge. Then RemoteFunctions, which return an answer, and why the server never waits on a client. It covers what survives the trip: copies instead of tables, no metatables, no functions, and only the Instances the receiver can see. It ends with UnreliableRemoteEvent: when a dropped message is fine, the 1,000-byte limit, and bandwidth as a budget.

## Lessons

1. [RemoteEvent](01-remote-events.md) — Fire a message from the client to the server and back, with the engine naming who sent it.
2. [RemoteFunction](02-remote-functions.md) — A call that returns: the client asks, waits, and gets the server's answer. And why it never runs the other way.
3. [What crosses the wire](03-what-crosses-the-wire.md) — Every argument arrives as a copy, and some arrive as nothing at all. Send plain data the other side can use.
4. [UnreliableRemoteEvent](04-unreliable-events.md) — Trade guaranteed delivery for speed on messages the next one replaces, and spend bandwidth like a budget.
