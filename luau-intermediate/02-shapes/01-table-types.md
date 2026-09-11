# Typing a table

*Field by field for records, an indexer for lists, and the difference that catches people out.*

Luau Intermediate · Typing Data · 10 min

---

Luau tables do two jobs and the type syntax has two shapes to match. A **record** has a fixed set of named fields and you write every one of them out. A **list or map** has keys you do not know in advance, and you describe the key and value types instead.

Getting this choice right is most of what typing data is. A record type is precise and catches typos; an indexer is flexible and catches nothing about which keys exist.

**A record type, field by field**

```luau
--!strict
type Item = {
	name: string,
	qty: number,
	tag: string?,
}

local lamp: Item = { name = "lamp", qty = 2, tag = "lighting" }
print(lamp.name, lamp.qty, lamp.tag)
```

Prints `lamp  2  lighting`. Fields are separated by commas, `?` marks the optional one, and a trailing comma is allowed.

**An array type, and an indexer**

```luau
--!strict
type Names = { string }              -- shorthand: an array of strings
type Counts = { [string]: number }   -- an indexer: any string key

local names: Names = { "lamp", "rope" }
local counts: Counts = { lamp = 2, rope = 1 }

print(#names, names[1], counts.lamp)
```

Prints `2  lamp  2`. `{ string }` is sugar for `{ [number]: string }`; you will write the shorthand nearly always and the longhand when the key is not a number.

The difference that catches people out: a record type lists **exactly** the fields it has. Reading `item.colour` when the type says `color` is an error, and so is a literal that leaves out a field the type requires. That precision is the point — it is what turns a silent `nil` into a report.

An indexer gives none of that. `counts.anything` is a `number` as far as the checker is concerned, whether or not that key was ever set. Use one when the keys really are open-ended, and a record when they are not.

## Read-only fields

A field can be marked `read`, which says callers may look at it and may not assign to it. It is how you describe a shape that is handed out but not handed over — a config table, an id that is set once at construction.

It is a checker-only guarantee: nothing stops a write at run time, exactly as nothing enforces any other annotation. What it buys you is that the write is reported rather than discovered.

**`read` on a field**

```luau
--!strict
type Config = {
	read id: string,
	retries: number,
}

local cfg: Config = { id = "store-1", retries = 3 }
cfg.retries = 5      -- fine
print(cfg.id, cfg.retries)
```

Prints `store-1  5`. Assigning to `cfg.id` would be reported as a write to a read-only property; the run would not care either way.

**Key points**

- A **record** type lists its fields exactly: a missing required field and an extra unlisted one are both errors.
- `{ T }` is an array of `T`; `{ [K]: V }` is the longhand indexer for open-ended keys.
- An indexer says nothing about which keys exist, so prefer a record wherever the fields are known.
- `?` on a field makes it optional, and the next lesson is about using one safely.
- `read` on a field forbids assignment — to the checker, not to the runtime.
