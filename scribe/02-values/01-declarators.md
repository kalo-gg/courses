# Declarators and bounds

*Int, Number, String, Enum, Optional and Dynamic: what each one enforces on a write, and what a clamp actually does.*

Scribe · Fields & Values · 10 min

---

A declarator says three things at once: the default, the Luau type, and the rule applied on every write. `Scribe.Int` rounds a non-integer write and clamps an out-of-range one; `Min` and `Max` must be finite integers or the template refuses to compile. `Scribe.String`'s `MaxLength` counts **bytes**, and an over-long write is cut on a character boundary. `Scribe.Enum`'s default must be a member.

**The scalar declarators, on Emberfall's fields**

```luau
local RARITIES = { "Common", "Rare", "Epic", "Legendary" }

Coins = Scribe.Int(0, { Min = 0 }),
Accuracy = Scribe.Number(0, { Min = 0, Max = 1, Precision = 0.001 }), -- 2 bytes on the wire
Nick = Scribe.String("", { MaxLength = 8 }), -- bytes, not characters
Rarity = Scribe.Enum("Common", RARITIES),
Nickname = Scribe.Optional(Scribe.String("", { MaxLength = 20 })), -- nil until written
```

## What a bound does on a write

Under the default `BoundsPolicy = "Clamp"`, `data.Coins.Set(-10)` stores `0`, returns `0`, and fires an anomaly: `OnAnomaly` on the server, and an `ANOMALY` warning with `Reason = "OutOfBounds"`. An over-long string is cut with `OverMaxLength`; an enum value outside the set cannot be clamped, so it is refused with an error and `NotAMember`.

`BoundsPolicy = "Reject"` throws at the call site instead, which is what you want while hunting the bad value. Functions, Instances, non-finite numbers, invalid UTF-8 and mixed array-and-dictionary tables are always refused.

**A clamp, seen from the call site**

```luau
print(data.Coins.Set(-10)) --> 0
--> [warn] [Scribe][Integrity][ANOMALY] implausible write at "Coins": OutOfBounds …

data.Nick.Set("Emberknight")
print(data.Nick.Get()) --> Emberkni (eight bytes)

print(pcall(function()
	return data.Rarity.Set("Mythic")
end)) --> false, an error naming the field: not a member
```

`Set` returns what was stored, not what you passed. Read the return value when the difference matters.

| Declarator | Default | Rule on a write | On the wire |
| --- | --- | --- | --- |
| `Scribe.Int` | an integer | rounds, clamps to Min and Max | narrow when bounded |
| `Scribe.Number` | a number | clamps; `Precision` narrows the client's copy | 8 bytes, or 1 to 4 with Precision |
| `Scribe.String` | a string | truncated at `MaxLength` bytes | length-prefixed |
| `Scribe.Enum` | a member | a non-member is refused | one byte |
| `Scribe.Optional` | none | nil until written; `Set(nil)` clears | absent, or the inner form |
| `Scribe.Dynamic` | the factory's result, per new profile | stored once, never overwritten | the inner form |

## Defaults computed per profile

A template default is evaluated once, when the module loads, so `CreatedUnix = os.time()` hands every new profile the server's start time. `Scribe.Dynamic(os.time)` takes the function itself and runs it once per profile with no stored value; a stored value is never overwritten.

The factory is called once at load to sample its type, so it must be pure, and it cannot see the player. Per-player seeding belongs in `OnPlayerInit`, in the Sessions module.

**Right and wrong creation stamps**

```luau
-- WRONG: every player's CreatedUnix is the server-start time, not their own.
CreatedUnix = os.time(),

-- Right: the function, not its result.
CreatedUnix = Scribe.Dynamic(os.time),
JoinedAt = Scribe.Dynamic(function()
	return DateTime.now()
end),
```

**Key points**

- A declarator carries a default, a Luau type and a rule applied on every write.
- Under Clamp, an out-of-range number is pinned and an anomaly fires; a non-member enum value is refused; Reject throws for all of them.
- `Set` returns what was stored, which is not always what you passed.
- `Scribe.Dynamic(fn)` runs a pure factory once per new profile; `os.time()` written directly is one timestamp for everyone.
