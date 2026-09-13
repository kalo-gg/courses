# Who sees a field

*Every root answers two questions, saved or not and seen by whom; ServerOnly, Shared and Session set the answers.*

Scribe · Replication & Commands · 10 min

---

By default a field is saved and its owner sees it. Wrapping a root changes the answers: `Scribe.ServerOnly` is saved and nobody sees it; `Scribe.Shared` is saved and every client sees it; `Scribe.Session` is never saved and resets on every rejoin. `AntiCheat` is absent from the client's accessor type, so reading it there is a type error rather than a nil at runtime.

**Three wrapped roots on Emberfall**

```luau
local template = {
	Coins = Scribe.Int(0, { Min = 0 }), -- saved, the owner sees it
	Gems = Scribe.Int(0, { Min = 0 }),

	Nameplate = Scribe.Shared({ -- saved, everyone sees it
		Title = Scribe.String("Wanderer", { MaxLength = 24 }),
	}),

	AntiCheat = Scribe.ServerOnly({ -- saved, nobody sees it
		Strikes = Scribe.Int(0, { Min = 0 }),
	}),

	Combat = Scribe.Session({ InCombat = false }), -- not saved, the owner sees it
}
```

| Wrapper | Persists? | Replicates to | Use for |
| --- | --- | --- | --- |
| none | yes | the owner | most player data |
| `Scribe.ServerOnly(v)` | yes | nobody | secrets, anti-cheat state |
| `Scribe.Shared(v)` | yes | every client | public info such as a title |
| `Scribe.Session(v)` | no | the owner | runtime-only state |

## Combining and nesting

Saving and replication are independent axes, so one wrapper from each combines: `ServerOnly(Session(…))` is runtime-only and server-side. Two from the same axis is a startup error. `Shared` and `Session` are root-only; `ServerOnly` may also wrap a nested field, or a field inside an element shape, which is then stripped from every entry on the way out. A wrapper goes outside the declarator, so `Scribe.ServerOnly(Scribe.DictOf(…))` is a whole server-side container.

**Wrapping declarators and element fields**

```luau
Aggro = Scribe.ServerOnly(Scribe.Session({ ThreatLevel = 0 })), -- runtime, server-side only
Badges = Scribe.Shared(Scribe.SetOf(Scribe.String("", { MaxLength = 32 }))), -- a public set
Stash = Scribe.ServerOnly(Scribe.DictOf({ -- a whole server-side container
	Qty = Scribe.Int(1, { Min = 1, Max = 999 }),
	Rarity = Scribe.Enum("Common", RARITIES),
}, { MaxKeys = 200 })),

Inventory = Scribe.DictOf({
	Qty = Scribe.Int(1, { Min = 1, Max = 999 }),
	Rarity = Scribe.Enum("Common", RARITIES),
	RollSeed = Scribe.ServerOnly(Scribe.Int(0)), -- never leaves the server, per entry
}, { MaxKeys = 200 }),
```

## What Session costs, and what Shared gives away

`Session` means not saved, not not sent: a Session root rides in the join snapshot like any other, so cap its containers; `ProfileSize` measures what is saved and cannot warn you. `Shared` broadcasts to every client: right for a public title, wrong for a spendable currency, whose *movement* alone tells everyone when someone spent. Publish a rank or a tier instead.

**Seen from the server, everything is there**

```luau
data.AntiCheat.Strikes.Increment(1)
data.Nameplate.Title.Set("Ember Knight")
data.Combat.InCombat.Set(true)

leave(ava)
task.wait(1)
local again = Data.WaitForData(join("Ava", 101))

print(again.AntiCheat.Strikes.Get(), again.Nameplate.Title.Get(), again.Combat.InCombat.Get())
--> 1  Ember Knight  false
```

**Key points**

- A root is saved and owner-visible by default; `ServerOnly`, `Shared` and `Session` change one answer each.
- One wrapper from each axis combines; two from the same axis is a startup error.
- `Shared` and `Session` are root-only; `ServerOnly` also works on nested and element fields.
- A Session root is still sent on join, so cap its containers; a Shared currency leaks its own movement.
