# Arrays and sets

*ArrayOf for order and eviction, SetOf for membership; which container to pick, and why an index is not an identity.*

Scribe · Containers & Big Numbers · 11 min

---

`Scribe.ArrayOf(shape, opts)` is an ordered list. `Insert(item, position?)` creates an entry, `Remove(index)` deletes one and returns it, and `data.RecentRuns[1].Score.Get()` reads by position. `MaxItems` makes an insert at the cap an error; `Evict = "Front"` or `"Back"` turns the cap into a rolling window that drops from the end you name. Appends put the oldest at the front, so `"Front"` is what appends want. The drop is a real removal: it fires `OnRemove` and replicates.

**The last runs, as a rolling window**

```luau
RecentRuns = Scribe.ArrayOf({
	Zone = Scribe.String("", { MaxLength = 32 }),
	Score = Scribe.Int(0, { Min = 0 }),
}, { MaxItems = 10, Evict = "Front" }),

-- later
data.RecentRuns.Insert({ Zone = "Ashfall Ridge", Score = 4200 })
data.RecentRuns[1].Score.Get()
local oldest = data.RecentRuns.Remove(1) -- by index, returns the removed entry
data.RecentRuns.Count()
```

## Sets

`Scribe.SetOf(element, opts)` is for membership rather than order: regions unlocked, quests completed. `Add` on a member and `Remove` on a non-member both return `false` and do nothing: no write, no replication, no `Changed`. That makes `Add` a safe idempotent grant whose return value answers "was this the first time". Entries stay deduplicated and sorted. The element must be a scalar; there is no `Insert`, no index `Remove`, no `Evict`.

**Regions unlocked**

```luau
Unlocked = Scribe.SetOf(Scribe.String("", { MaxLength = 32 }), { MaxItems = 64 }),

data.Unlocked.Add("AshfallRidge") --> true
data.Unlocked.Add("AshfallRidge") --> false, already a member
data.Unlocked.Has("AshfallRidge") --> true
data.Unlocked.Remove("AshfallRidge") --> true
data.Unlocked.Count() --> 0
data.Unlocked.Clear()

if data.Unlocked.Add(regionId) then
	data.Xp.Increment(250) -- only the first time
end
```

## Positions are not identities

`data.RecentRuns[2]` means whatever sits at index 2 right now. A per-element `Observe` follows the index: remove below it and the listener fires with whatever moved into the slot. Follow elements with `OnInsert` and `OnRemove` instead. `Has`, `Find` and `RemoveValue` compare by value, so the entry `Get()` handed you finds the stored one. That is also why `Scribe.Timed` and `Scribe.Dynamic` are refused inside an element shape.

**Removing by value**

```luau
local run = data.RecentRuns.Get()[2]
data.RecentRuns.RemoveValue(run) -- removes index 2, returns the entry
data.RecentRuns.Find(run) --> nil now

-- data.RecentRuns[2].Set(nil) is refused except on the last entry:
-- a hole would split #array from Count()
```

> **Two write rules for every array**
>
> `Insert(nil)` and a non-integer position are errors. A table that mixes array indices with string keys is rejected at the write, because the DataStore's JSON encoder could never save it, and that used to surface as a lost save long after the write.

**Key points**

- `Scribe.ArrayOf`: `Insert`, `Remove(index)` returns the entry, `MaxItems` caps, `Evict` names the end to drop from.
- `Scribe.SetOf`: `Add` and `Remove` return false when there is nothing to do, which makes `Add` a safe first-time gate.
- An index is a position, not an identity; follow elements with `OnInsert` and `OnRemove`.
- `Has`, `Find` and `RemoveValue` compare by value.
