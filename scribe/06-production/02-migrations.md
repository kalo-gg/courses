# Migrations

*Evolve your own template with a numbered chain that fails closed, and key each step off something reconcile cannot fake.*

Scribe · Production Data · 11 min

---

When the template changes shape, `Migrations = { [2] = function(data) … end, [3] = … }` runs during the load, between reconcile and `OnPlayerInit`, against the raw stored table. It fails closed: a step that throws ends the session with a kick and `migration-failed`, and nothing is stamped or saved. Each profile stores the version its last successful run stamped, so never renumber or remove a shipped step, and keep the table contiguous from 2.

**Bag became Inventory**

```luau
Migrations = {
	[2] = function(data)
		if data.Bag ~= nil then
			data.Inventory = {}
			for _, itemId in data.Bag do
				data.Inventory[itemId] = { Qty = 1, Rarity = "Common" }
			end
			data.Bag = nil
		end
	end,
},
```

## Running one in the sandbox

Two bundles in one script: the first, with `Bag = {}` in its template, joins Ava, writes her bag, and she leaves; `Data.Stop()` releases it. The second declares `Inventory` and the step above. On the next join the step's own print shows the bag it moved, and the accessor reads the new shape. Keys the new template no longer knows are left in place with a warning: removing them is a migration's job, which is what `data.Bag = nil` does.

**v1 writes a bag, v2 migrates it**

```luau
local v1 = Scribe({ Template = { Coins = Scribe.Int(100, { Min = 0 }), Bag = {} }, … }).Server
local d1 = v1.WaitForData(join("Ava", 101))
d1.Bag.Set({ "Torch", "Torch" })
leave(ava)
task.wait(1)
v1.Stop()

local v2 = Scribe({
	Template = { Coins = Scribe.Int(100, { Min = 0 }), Inventory = Scribe.DictOf({ Qty = Scribe.Int(1, { Min = 1 }) }) },
	Migrations = {
		[2] = function(data)
			print("migrating", data.Coins, "has Bag", data.Bag ~= nil) --> migrating 100 has Bag true
			-- move the bag, then data.Bag = nil
		end,
	},
	…
}).Server
local d2 = v2.WaitForData(join("Ava", 101))
print(d2.Inventory.Torch.Qty.Get()) --> 2
```

> **A near-empty account is not a test**
>
> A returning profile still holding nothing but defaults is loaded un-migrated and unstamped rather than kicked, so "my account loaded fine" says nothing about whether the step ran on a profile with real progress.

## Shipping one

A rolling deploy bounces players: a profile a new server migrated can land on an old server, which fails closed with `VersionAheadPolicy = "Kick"` and `VersionAheadMessage`; shut old servers down when you ship a migration. `MigrationShadow = true` re-runs the chain against the raw pre-reconcile bytes and warns where the two diverge, for a staging deploy only. A step's edits to `_Scribe` are discarded and reported as `MIGRATION_RESERVED_DISCARDED`, so it cannot destroy receipt idempotency by accident.

**A chain, and a stored field that became derived**

```luau
Migrations = {
	[2] = function(data) -- Bag -> Inventory
		-- …
	end,
	[3] = function(data)
		-- Level used to be stored; now it is Scribe.Derived from Xp.
		-- Reconcile leaves the old key behind, so a step deletes it.
		data.Level = nil
	end,
},
VersionAheadPolicy = "Kick",
VersionAheadMessage = "Emberfall is updating. Rejoin in a moment.",
```

**Key points**

- `Migrations` is a contiguous chain from 2; a throwing step kicks with `migration-failed` and saves nothing.
- Reconcile runs first, so key each step off a field you removed, never off a missing template key.
- A profile holding only defaults is loaded unstamped, so test on an account with real progress.
- Ship a migration with old servers shut down; `VersionAheadPolicy = "Kick"` covers the gap.
- A step cannot touch `_Scribe`; its edits there are discarded and logged.
