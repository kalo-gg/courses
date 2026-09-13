# The Emberfall template

*Read the docs' example template line by line: plain values, declarators, nested tables and the three options every bundle needs.*

Scribe · Scribe Foundations · 10 min

---

A **template** is a plain Luau table describing one player's data. Scribe compiles it once at startup, and typing, validation, wire packing and saving are all driven from it. Emberfall's is below, and every guide in the docs shows a slice of it.

**ReplicatedStorage/Shared/EmberfallData.luau**

```luau
local Scribe = require(game:GetService("ReplicatedStorage").Packages.Scribe)

local RARITIES = { "Common", "Rare", "Epic", "Legendary" }

-- Pure: no yields, no randomness, no os.time. Scribe runs it on both realms.
local function levelForXp(xp: number): number
	return math.min(100, math.floor(xp / 1000) + 1)
end

local template = {
	Coins = Scribe.Int(0, { Min = 0 }),
	Gems = Scribe.Int(0, { Min = 0 }),
	Xp = Scribe.Int(0, { Min = 0 }),
	Level = Scribe.Derived(Scribe.Int(1, { Min = 1, Max = 100 }), { "Xp" }, levelForXp),
	Inventory = Scribe.DictOf({
		Qty = Scribe.Int(1, { Min = 1, Max = 999 }),
		Rarity = Scribe.Enum("Common", RARITIES),
	}, { MaxKeys = 200 }),
	Settings = Scribe.Flags({ "Music", "Sfx", "TutorialDone" }),
	LastDaily = Scribe.Timed(false),
	Stats = {
		Deaths = Scribe.Int(0, { Min = 0 }),
		Playtime = Scribe.Int(0, { Min = 0 }),
	},
}

return Scribe({ Template = template, ProfileStoreIndex = "EmberfallPlayerData", ProfileKeyPrefix = "PLAYER_" })
```

## Reading it line by line

| Line | What it declares | Covered in |
| --- | --- | --- |
| `Scribe.Int(0, { Min = 0 })` | A whole number that cannot go below zero | Declarators |
| `Scribe.Derived(Scribe.Int(1, …), { "Xp" }, levelForXp)` | A read-only field computed from Xp | Derived fields |
| `Scribe.DictOf({ … }, { MaxKeys = 200 })` | A dictionary of records keyed by item id, capped | Dictionaries |
| `Scribe.Flags({ "Music", "Sfx", "TutorialDone" })` | Three named booleans in one field | Flags and big numbers |
| `Scribe.Timed(false)` | A value that clears itself back to false | Timers and cooldowns |
| `Stats = { … }` | A plain nested table of ordinary fields | This lesson |

Plain values work: `Playtime = 0` would be a number field defaulting to zero, with no rules attached. A **declarator** adds the rules, and it carries three things at once: the default a new profile starts with, the Luau type your code sees, and the runtime metadata Scribe applies on every write.

`Stats` is a plain nested table. Scribe walks into it and treats `Deaths` and `Playtime` as ordinary fields one level down.

## The three options every bundle needs

`Template`, `ProfileStoreIndex` (the DataStore name) and `ProfileKeyPrefix` have no defaults, on purpose: a default would let two games, or a test build and a live one, silently share one store. Ava's key is the prefix and her user id: `PLAYER_101`. An empty prefix is allowed and means the bare user id.

Every other option has a default. Luau accepts unknown table keys, so a misspelled option is caught in Studio with an `UNKNOWN_OPTION` warning rather than a type error.

**The call, annotated**

```luau
return Scribe({
	Template = template, -- the shape of one player's data
	ProfileStoreIndex = "EmberfallPlayerData", -- the DataStore profiles live under
	ProfileKeyPrefix = "PLAYER_", -- Ava's key is PLAYER_101
})

-- on the server
local Data = require(ReplicatedStorage.Shared.EmberfallData).Server
```

> **Name your store deliberately**
>
> Changing `ProfileStoreIndex` or the prefix on a live game hides every existing profile. A fresh name is how you want a test build isolated, and how you accidentally wipe production.

**Key points**

- A template is a plain table; Scribe compiles it once and drives everything from it.
- A declarator carries a default, a Luau type and the runtime rule applied on every write.
- Plain values and nested tables are fine where a field needs no rules.
- `Template`, `ProfileStoreIndex` and `ProfileKeyPrefix` are required; naming your own store is deliberate.
