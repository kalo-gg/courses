# Tables inside tables

*Rooms with exits, players with inventories: nesting is how real data is shaped, and how it breaks.*

Luau Fundamentals · Tables · 10 min

---

A table can hold another table, and that is how every real data structure you will build is shaped: rooms that each have exits, players that each have an inventory, a save file that has a section per system.

Nothing new is needed for this. A nested table is just a value, and `rooms.hall.exits.north` is three ordinary lookups in a row.

**A rooms table, nested**

```luau
local rooms = {
	hall = {
		name = "the hall",
		description = "A wide hall with a cold floor.",
		exits = { north = "library" },
	},
	cellar = {
		name = "the cellar",
		description = "Damp, and darker than it should be.",
	},
}

print(rooms.hall.name)
print(rooms.hall.exits.north)
```

Prints `the hall` and then `library`. This is the shape both of this course's projects use: a table of rooms, each a record, some of them holding an `exits` table of direction to room key.

## Reading two levels down safely

One level of missing data is free — `rooms.attic` is `nil` and nobody complains. Two levels is where it breaks, because the second lookup happens on a `nil`.

There are two ways to make it safe, and both are worth knowing:

- **Check the level above** with an `if`, then read. Clearest when you have something useful to do about the absence.
- **Chain with `and`**, which short-circuits: `rooms.attic and rooms.attic.exits and rooms.attic.exits.north` stops at the first `nil` and gives you `nil` instead of an error.

The chain is the one you will write most, because most of the time the answer really is just "then there is no exit".

**Two ways to guard**

```luau
-- 1. Check, then read
local exit
if rooms.cellar and rooms.cellar.exits then
	exit = rooms.cellar.exits.north
end
print(exit) -- nil, and no error

-- 2. Chain with and, which stops at the first nil
local north = rooms.cellar and rooms.cellar.exits and rooms.cellar.exits.north
print(north) -- nil, and no error

-- The chain gives you the value when everything is there
local fromHall = rooms.hall and rooms.hall.exits and rooms.hall.exits.north
print(fromHall) -- library
```

`and` returns its first falsy operand, so a chain of them returns `nil` the moment any link is missing — and the final value when none is. Every guard in both of this course's projects is one of these two shapes.

> **Tables are references**
>
> `local b = a` on a table does not copy it. Both names now point at the *same* table, so `b[1] = 99` is visible through `a` as well. This is the exception to "assignment copies the value" from Module 1 — the value being copied is the reference. `table.clone(a)` makes a real copy, one level deep: the top-level keys are new, but a nested table inside is still shared.

That reference behaviour is not a wart; it is what makes nested data cheap. Passing a rooms table around costs nothing because nothing is copied. It only bites when you meant to take a snapshot — and then `table.clone` is the answer, with the caveat that a second `clone` is needed for each level you actually want separated.

**Key points**

- A table holding tables is how real data is shaped; a nested lookup is just several lookups in a row.
- One missing level is `nil`; two is `attempt to index nil with '<the key you asked for>'`.
- Guard with an `if` on the level above, or chain with `and`, which short-circuits.
- `local b = a` shares one table. `table.clone` copies it, one level deep.
