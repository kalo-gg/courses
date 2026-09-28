# 4. Players

*Players joining and leaving, the characters they drive, the Humanoid, and where per-player state lives.*

Roblox Fundamentals · Medium

---

A Player is the connection, and the character is the body it drives. This module covers players joining and leaving, including the ones who arrive before your script does; characters, which are rebuilt on every respawn, and the way back from a touched part to its player; the Humanoid's health, ForceFields and movement; and the three homes of per-player state (leaderstats, attributes and your own tables), plus teams. It ends with the obby's first project: lava, checkpoints and a Stage leaderstat.

## Lessons

1. [Joining and leaving](01-joining-and-leaving.md) — PlayerAdded and PlayerRemoving, the players who were there before your script, and which name to trust.
2. [Characters](02-characters.md) — A character is a Model rebuilt on every spawn: handle each one as it arrives, and find the player from any part of it.
3. [The Humanoid](03-the-humanoid.md) — Health and the ForceField, death and respawning, and the movement properties, including which jump property counts.
4. [Player state and teams](04-player-state-and-teams.md) — leaderstats for what players see, attributes for what other scripts read, tables for the rest, and the Teams service.
