# 7. Shipping multiplayer

*Hiding latency without trusting the client, a codebase split cleanly across both sides, testing with many players, and client performance.*

Roblox Intermediate · Hard

---

This module gets the arena ready for real players on real connections. It hides latency without handing the client control: effects show at once, the server checks each hit against its own view with a tolerance, and `GetServerTimeNow` gives every client the same clock. It splits the code into Shared, Server and Client folders around one module that owns every remote, and names the point where Blink takes over. It tests with Studio's Server & Clients mode and the Network Simulator, which find the bugs a solo Test cannot. It ends with client performance: small `PreRender` work, lean interfaces, and effects taken from a pool. The capstone builds the arena blaster, from key press to server-checked hit.

## Lessons

1. [Latency and feel](01-latency-and-feel.md) — Show the shot on the client at once, let the server decide what it hit, and give every client the same clock.
2. [Structuring client and server](02-structuring-client-and-server.md) — Shared, Server and Client folders, one module that owns every remote, typed payloads, and when Blink takes over.
3. [Testing with many players](03-testing-multiplayer.md) — Server & Clients for bugs between players, the Network Simulator for bad connections, and a checklist for both.
4. [Client performance](04-client-performance.md) — Keep PreRender work small, keep busy interface apart from the rest, and reuse effects from a pool.
