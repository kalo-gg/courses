# The Humanoid

*Health and the ForceField, death and respawning, and the movement properties, including which jump property counts.*

Roblox Fundamentals · Players · 11 min

---

A lava floor calls `humanoid:TakeDamage(100)` on whatever touches it. In testing it kills everyone except players who have just spawned: for ten seconds they walk across it untouched.

That is not a bug in the lava. It is the spawn ForceField doing its job, and knowing which Humanoid calls respect it is most of this lesson.

## Health, and two ways to lower it

```luau
local humanoid = character:WaitForChild("Humanoid") :: Humanoid

humanoid:TakeDamage(20) -- does nothing while a ForceField protects the character
humanoid.Health -= 20   -- a ForceField does not stop this
humanoid.Health = 0     -- dead, whatever protects them
```

`TakeDamage` is ordinary damage, and a ForceField blocks it. Writing `Health` goes around every ForceField, so use it for lava and heals.

The docs say `Health` stays between 0 and `MaxHealth` (100 by default). At 0 the humanoid dies, `Died` fires, and its `Health` is locked at 0. A living player character also regenerates 1% of `MaxHealth` each second by default.

The ForceField comes from the SpawnLocation: `Duration` seconds of it, 10 by default, and none at 0. A negative `TakeDamage` heals, but a ForceField blocks that too.

## Movement

`WalkSpeed` is in studs per second, 16 by default. Jumping has two properties, and `UseJumpPower` picks the one that counts: `JumpPower`, an upward force, when it is `true`, and `JumpHeight`, in studs, when it is `false`. Neither is deprecated. Each spawn copies `StarterPlayer.CharacterUseJumpPower`, which defaults to `true`, but a place can change it.

```luau
humanoid.WalkSpeed = 24 -- studs per second; 16 by default

if humanoid.UseJumpPower then
	humanoid.JumpPower = 60 -- an upward force; 50 by default
else
	humanoid.JumpHeight = 9 -- in studs; 7.2 by default
end
```

The docs warn that the client can change `WalkSpeed`, so never use it as a security check.

| Respawning | Default | What it does |
| --- | --- | --- |
| `Players.RespawnTime` | 5 | Seconds from death to the new character, when characters load automatically. |
| `Players.CharacterAutoLoads` | `true` | With `false`, no character spawns, even on join, until you call `LoadCharacterAsync`. |
| `player:LoadCharacterAsync()` |  | Builds a new character now and removes the old one. It yields, and it clears the Backpack and PlayerGui. |

> **Recognising older code**
>
> `player:LoadCharacter()` is deprecated. `player:LoadCharacterAsync()` replaced it and loads the character the same way.

**Key points**

- `TakeDamage` respects a ForceField; writing `Health` ignores it. Use `Health` for lava and heals.
- A SpawnLocation gives `Duration` seconds of ForceField, 10 by default and none at 0.
- `Health` stays between 0 and `MaxHealth`; at 0 `Died` fires, and by default a new character spawns `RespawnTime` seconds later.
- `UseJumpPower` picks `JumpPower` (`true`) or `JumpHeight` (`false`); neither is deprecated.
- `LoadCharacterAsync` spawns a character now; `LoadCharacter` is its deprecated predecessor.
