# Lists

*Tables with number keys starting at 1, the length operator, and inserting and removing.*

Luau Fundamentals · Tables · 10 min

---

Luau has exactly one container type: the **table**. Everything a language elsewhere calls an array, a list, a dictionary, a map, a set, an object or a struct is a table here, and which of those it is depends entirely on the keys you put in it.

Start with number keys. `{ "ada", "bob", "cy" }` is a table whose keys are 1, 2 and 3 — **not** 0, 1 and 2. Lua counted from 1 forty years ago and Luau kept it, which means every off-by-one bug in ported code comes from exactly here.

**A list, and its length**

```luau
local names = { "ada", "bob", "cy" }

print(names[1], names[2], names[3])
print(#names)
print(names[4]) -- past the end: nil, not an error
```

Prints `ada bob cy`, then `3`, then `nil`. Reading past the end of a list is quiet, exactly like reading a key that was never set — the two are the same operation.

## Adding and removing

**table.insert and table.remove**

```luau
local queue = { "ada" }

table.insert(queue, "bob") -- append
table.insert(queue, "cy")
print(#queue, table.concat(queue, ", "))

local last = table.remove(queue)     -- from the end
local first = table.remove(queue, 1) -- from the front, shifting the rest down
print(last, first, table.concat(queue, ", "))
```

Prints `3` and `ada, bob, cy`, then `cy`, `ada`, `bob`. `table.remove` hands you the value it took out. Appending and removing from the end are cheap; inserting or removing at the front has to shift everything after it.

`#t` is not a count. It is a **border**: some index `n` where `t[n]` is not `nil` and `t[n + 1]` is. On a list with no gaps that is the count, which is why it looks like one.

Put a `nil` in the middle and there are several valid borders, and the language does not promise which one you get. That is not a bug to be worked around; it is the definition. Keep your lists gap-free — remove with `table.remove` rather than by assigning `nil` — and `#` is exactly the count you wanted.

One more habit worth forming now: a list you are going to remove from while walking it is a trap. Counting up with `for i = 1, #list` and calling `table.remove(list, i)` inside skips the element that shifted down into the slot you just left. Walk **backwards** — `for i = #list, 1, -1` — and the shifting happens behind you where it cannot hurt.

**Key points**

- One container type — the table — and number keys starting at **1** make it a list.
- Reading past the end is `nil`, not an error.
- `#t` is a border, not a count: it is only the count when the list has no gaps.
- `table.insert` appends and `table.remove` closes the gap; walk backwards when removing inside a loop.
