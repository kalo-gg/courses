# What a module exposes

*Everything not in the returned table is private, and that is the only privacy Luau has.*

Luau Intermediate · Structuring a Codebase · 10 min

---

You already know the mechanics: a file returns one value, and a caller gets that value back from `require`. The interesting question is not how, it is **what** — which names go in the returned table and which stay above it.

That choice is the only access control Luau has. There is no `private` keyword and no convention the language enforces. A local above the returned table simply cannot be named from anywhere else, and that is the whole story.

**A module with a private helper**

```luau
--!strict
local Store = {}

-- Private by construction: nothing outside this file can name it.
local function normalize(name: string): string
	return string.lower(name)
end

function Store.add(items: { [string]: number }, name: string, qty: number)
	local key = normalize(name)
	items[key] = (items[key] or 0) + qty
end

local items: { [string]: number } = {}
Store.add(items, "Lamp", 2)
Store.add(items, "lamp", 1)
print(items.lamp)
```

Prints `3` — both spellings normalise to the same key. `normalize` is a file-local, so it can be renamed, rewritten or deleted without any caller noticing. `Store.add` cannot.

That asymmetry is worth being deliberate about. Everything in the returned table is a promise: someone will call it, and changing it will break them. Everything above the table is yours.

So the default should be private. Put a function in the table when a caller genuinely needs it, not because it might one day be useful — the one that might be useful is the one you cannot delete two months later.

## Types cross the boundary too

**`export type` beside the functions**

```luau
--!strict
-- store.luau
export type Item = { name: string, qty: number }
export type Store = { items: { Item } }

local M = {}

function M.new(): Store
	return { items = {} }
end

function M.count(store: Store): number
	return #store.items
end

return M
```

A caller writes `local Store = require("./store")` and then annotates with `Store.Store` and `Store.Item`. Without the two exports, the same caller has to re-declare both shapes, and its copy drifts the first time this file changes.

The rule is simple: if a function's signature mentions a shape, export the shape. A module that returns `new()` without exporting the type of what `new` returns has only exported half an API.

And export the shapes only. A type used purely inside the module — the intermediate table a helper builds — stays a plain `type`, for exactly the same reason `normalize` stays a local.

**Key points**

- The returned table is the API; everything above it is private by construction, and that is Luau's only access control.
- Default to private — a function in the table is a promise you cannot take back cheaply.
- If a signature mentions a shape, `export type` the shape, or callers re-declare it and their copies drift.
- Types used only inside the module stay plain `type`, exactly as private helpers stay locals.
- Half an exported API — functions without their types — makes every caller do work you could have done once.
