# Records

*String keys, dot against bracket access, and what reading a key that was never set gives you.*

Luau Fundamentals · Tables · 9 min

---

Give a table names instead of numbers and it becomes a **record**: a bundle of labelled fields, the thing another language would call an object or a struct. Nothing about the table changed — you are just using a different kind of key.

A table can hold both halves at once, and real code often does: a list of items with a `count` field beside them is perfectly ordinary.

**A record**

```luau
local player = {
	name = "Ada",
	level = 7,
	["favourite colour"] = "green",
}

print(player.name, player["level"])
print(player["favourite colour"])
print(player.title) -- never set
```

Prints `Ada 7`, then `green`, then `nil`. `name = "Ada"` inside the braces is shorthand for the key `"name"`; a key that is not a plain identifier needs the bracket form on both sides.

`player.name` and `player["name"]` are **exactly the same operation**. The dot is sugar for the bracket with a literal string in it.

You need brackets in two cases: when the key is not a valid identifier (it has a space, starts with a digit, or is a reserved word like `end`), and when the key lives in a variable. That second one is the important one — `player[field]` looks up whatever string `field` holds, while `player.field` looks up the literal five letters `field`.

## Absent, not empty

Reading a key that was never set gives `nil`. Assigning `nil` to a key **removes** it. Those two facts together have a consequence worth sitting with: there is no way to tell "never set" apart from "set to `nil`". A table does not remember keys it does not have.

This is also why the default-value idiom from Module 2 is everywhere in table code: `local title = player.title or "no title"` handles the missing field in one line, as long as `false` is not a value the field could legitimately hold.

**Removing a key**

```luau
local player = { name = "Ada", level = 7 }

player.level = nil

print(player.level)    -- nil: removed
print(player.neverSet) -- nil: never there

local title = player.title or "no title"
print(title)
```

The two `nil`s are indistinguishable, and that is by design. Printing `no title` at the end is the `or` default doing the work an `if` would otherwise do.

**Key points**

- String keys make the same table a record; a table can be a list and a record at once.
- `t.name` is exactly `t["name"]`; brackets are for computed keys and awkward names.
- A key that was never set reads as `nil`, and assigning `nil` removes a key.
- `t.field or default` fills in a missing field in one line.
