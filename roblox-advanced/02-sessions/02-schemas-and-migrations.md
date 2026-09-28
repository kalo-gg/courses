# Schemas and migrations

*Bring every old save up to date as it loads: a template for new fields, and numbered migration steps, in order, for changed ones.*

Roblox Advanced · Sessions · 11 min

---

Your first release saved `{ Gold = 120, Items = { "Sword", "Sword" } }`. Version 3 of the game calls it `Coins` and counts items: `Inventory = { Sword = 2 }`. A player back after months loads the old shape, and the code reads `data.Coins` as `nil`. Treat that as 0 and the next save writes 0 over their gold.

Old saves never change by themselves. Each load has to bring them up to date: a template for fields that are new, and migrations for fields that changed.

## Reconcile with a template

```luau
local TEMPLATE = {
	Coins = 0,
	Inventory = {},
	Settings = { Music = true, Sfx = true },
}

-- Fills in what `data` is missing from `template`, at every depth. Removes nothing.
local function reconcile(data, template)
	for key, default in template do
		if type(default) == "table" then
			if data[key] == nil then
				data[key] = {}
			end
			if type(data[key]) == "table" then
				reconcile(data[key], default)
			end
		elseif data[key] == nil then
			data[key] = default
		end
	end
	return data
end
```

A table default is built fresh for each player, never shared. Keys the template does not list are left alone.

A template cannot rename `Gold` or turn a list into counts. For that, save a `Version` number and keep one migration step per version. On load, run every step above the saved version, lowest first, then reconcile. A save from before you added `Version` has none: treat it as 1.

## Migrations, in order

```luau
local LATEST = 3
local MIGRATIONS = {
	[2] = function(data) -- v2 renamed Gold to Coins
		data.Coins, data.Gold = data.Gold, nil
		return data
	end,
	[3] = function(data) -- v3 counts items instead of listing them
		local inventory = {}
		for _, name in data.Items or {} do
			inventory[name] = (inventory[name] or 0) + 1
		end
		data.Inventory, data.Items = inventory, nil
		return data
	end,
}

local function migrate(data)
	for version = (data.Version or 1) + 1, LATEST do
		data = MIGRATIONS[version](data)
		data.Version = version
	end
	return data
end
```

Count from the saved version, never `for _, step in MIGRATIONS`: a loop over the table promises no order for these keys, and would re-run steps the save already had. A field leaves only in a step, on purpose, like `Gold` here.

Notice what never happens: no field is deleted just because the template does not list it. It may come from a newer server or a feature still in testing, and a load that drops it deletes that data at the next save.

> **A save from a newer server**
>
> While an update rolls out, old and new servers run side by side. A player can save on a new server as version 4, then land on an old one that knows only 3. Refuse to load that save and ask the player to rejoin: an old server that loads it would save it back without knowing what version 4 means.

**Key points**

- Old saves keep their old shape until a load changes it, so every load brings them up to date.
- Reconcile fills missing fields from a template, at every depth, and never removes anything.
- Save a `Version`, and run every migration step above it in order, counting up; treat a save with no `Version` as version 1.
- A field disappears only in a migration step that removes it on purpose.
- Refuse a save whose `Version` is newer than this server knows.
