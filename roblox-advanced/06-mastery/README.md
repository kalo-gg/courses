# 6. Running a live game

*Physics exploits, text and safety rules, moderation, and keeping a game up through updates.*

Roblox Advanced · Hard

---

A game that works still has to survive its players and its own updates. This module takes physics back from clients where the game depends on it and checks character movement on the server without flagging honest players. It filters every piece of text one player writes for others, bans rule-breakers with the ban API and logs each action for appeals, then releases updates with Restart Servers, changes values live with experience configs and reports errors from live servers. The capstone that follows builds the hub around the profile store.

## Lessons

1. [Physics and exploits](01-physics-and-exploits.md) — Take physics back from clients where the game depends on it, and catch impossible movement without flagging honest players.
2. [Text, chat and safety](02-text-and-safety.md) — Filter every piece of text a player writes for others, once and on the server, and check what local rules allow each player.
3. [Moderation](03-moderation.md) — Keep a rule-breaker out with the ban API, not just this server, and log every action so appeals can be answered.
4. [Keeping a game up](04-keeping-a-game-up.md) — Move players to a new version with Restart Servers, change values without one, and hear about errors on live servers.
