# Aliases and export type

*Name a shape once with type, share it between files with export type, and give it parameters.*

Luau Intermediate · Typing Data · 10 min

---

You have been writing aliases since the first lesson of this module. `type Item = { … }` gives a shape a name; everywhere that name appears the checker substitutes the shape.

The reason to bother is not brevity. It is that one alias is one place to change, and that two files using the same alias genuinely agree — where two files writing out the same record type by hand only agree until one of them is edited.

**Aliases built out of aliases**

```luau
--!strict
type Item = { name: string, qty: number }
type Store = { items: { Item }, capacity: number }

local store: Store = {
	items = { { name = "lamp", qty = 2 } },
	capacity = 5,
}

print(#store.items, store.capacity)
```

Prints `1  5`. `{ Item }` is an array of the record, and `Store` is the shape the rest of a program passes around instead of the four lines it stands for.

## Across a module boundary

**`export type` and how a caller reaches it**

```luau
--!strict
-- types.luau
export type Item = { name: string, qty: number }
export type Store = { items: { Item } }

return {}

-- main.luau
-- local types = require("./types")
-- local lamp: types.Item = { name = "lamp", qty = 2 }
```

`export` makes an alias visible outside its file. The caller reaches it through the name it required the module under — `types.Item` — and a module whose only job is types still has to return something.

This is how a codebase agrees on one shape. A module that returns functions but not the types describing them forces every caller to write the record out again, and those copies drift.

Export the type beside the function that produces it. If `Store.new()` returns a store, `Store.Store` should be the type of one.

Aliases take **parameters**, which is what makes a shape reusable across element types: `type Box<T> = { value: T }` and then `Box<number>`, `Box<string>`.

And unlike a generic function, a generic alias may give a parameter a **default**. `type Result<T, E = string>` means `Result<number>` is legal and means `Result<number, string>`. Functions may not do this; the Generics & Functions module comes back to why.

**A parameter, and a parameter with a default**

```luau
--!strict
type Box<T> = { value: T }
type Result<T, E = string> = { ok: true, value: T } | { ok: false, err: E }

local b: Box<string> = { value = "lamp" }
local found: Result<number> = { ok = true, value = 4 }
local missing: Result<number> = { ok = false, err = "no such item" }

print(b.value, found.ok, missing.ok)
```

Prints `lamp  true  false`. `Result<number>` fills `E` from the default, and `{ ok: true, … } | { ok: false, … }` is a tagged union whose tag is a boolean literal.

**Key points**

- `type Name = Shape` names a shape; the checker substitutes it wherever the name appears.
- `export type` makes the alias visible to other files, reached as `alias.Name` through the require.
- Export the type beside the function that produces the value, or every caller re-declares the shape.
- Aliases take parameters — `type Box<T>` — and, unlike functions, may give them **defaults**.
- A module of nothing but exported types still has to return a value.
