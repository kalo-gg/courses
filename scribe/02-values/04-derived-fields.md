# Derived fields

*Declare a formula once and read the answer like any other field; nothing ever writes it and nothing can drift.*

Scribe · Fields & Values · 11 min

---

A player's level is their XP through a formula, so storing it stores the same fact twice. `Scribe.Derived(output, inputs, compute)` declares the formula once. The field has no `Set`, `Update` or `Increment`: they are absent from its type, and throw if reached anyway. One source of truth, and changing the function fixes every player at once.

**Level from Xp**

```luau
local function levelForXp(xp: number): number
	return math.min(100, math.floor(xp / 1000) + 1)
end

local template = {
	Xp = Scribe.Int(0, { Min = 0 }),
	Level = Scribe.Derived(Scribe.Int(1, { Min = 1, Max = 100 }), { "Xp" }, levelForXp),
}

-- later
data.Xp.Increment(2500)
print(data.Level.Get()) --> 3
```

## Declaring one

`output` is an ordinary declarator supplying the type and bounds, which is why `Level` can cap itself at 100. `inputs` are dotted paths to declared fields (`"Stats.Deaths"` works), in the order `compute` receives them; a path that does not resolve is a startup error. `compute` is pure: no yields, writes, `os.time` or `math.random`. It runs once per realm at startup to seed the default.

A derived field may read another one; cycles are rejected. It must be a root, or sit inside a `Scribe.Session` root: never under a persisted record, because it must never be saved.

| Allowed as an input | Not allowed |
| --- | --- |
| `Scribe.Int`, `Scribe.Number`, `Scribe.String`, `Scribe.Enum`, a boolean | a record or a container such as `Inventory` |
| `Scribe.Big`, which arrives as the big value object | `Scribe.Flags`, whose read shape is a mutable list |
| a datatype field such as `Scribe.Vector3` | a `buffer` |
| `Scribe.Optional(…)`, where the argument may be nil | `Scribe.Timed`, which lapses on the clock, not on a write |
| another derived field | a container entry, or any runtime key |

**Chaining, and a projection of something the client cannot see**

```luau
Level = Scribe.Derived(Scribe.Int(1, { Min = 1, Max = 100 }), { "Xp" }, levelForXp),
Rank = Scribe.Derived(Scribe.String(""), { "Level" }, rankForLevel), -- settles after Level

Suspicion = Scribe.ServerOnly(Scribe.Int(0, { Min = 0, Max = 100 })),
-- The client cannot see Suspicion, so the server computes this one and sends it.
Watchlisted = Scribe.Derived(false, { "Suspicion" }, function(s)
	return s >= 80
end),
```

When the client can see every input it computes the field itself and nothing crosses the wire; a field reading something it cannot see is computed on the server and sent.

## When it recomputes

A recompute is an ordinary write: it validates against the output declarator, an unchanged result fires nothing, and `Changed` and `Observe` behave as usual. A `Set` or `Increment` on an input recomputes at once; a `Data.Batch` recomputes once at the end; a `Data.Transaction` once inside it; a client diff frame once per frame; a load once, before Ready.

Reads settle first, so a derived field never hands you a value its inputs have moved past, even mid-batch.

**Reads settle first, even mid-batch**

```luau
Data.Batch(player, function()
	data.Xp.Set(4200)
	data.Stats.Deaths.Increment(1)
	print(data.Level.Get()) --> 5, not the pre-batch level
end)
```

**Key points**

- `Scribe.Derived(output, inputs, compute)`: a declarator for the type and bounds, dotted input paths, a pure function.
- A derived field has no mutators; writing it throws, and the type checker catches it first.
- Inputs are declared leaves: numbers, strings, enums, booleans, bigs, datatypes, optionals and other derived fields.
- It recomputes on every input write, once per batch or transaction, and reads settle first.
