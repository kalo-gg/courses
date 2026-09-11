# The table library and generic for

*concat, sort, find, clone and freeze, and the for loop that walks a table you did not count.*

Luau Fundamentals · Tables · 10 min

---

The table library is nine functions long and you have already met two of them. The rest are worth knowing by name, because each one replaces a loop you would otherwise write by hand — and a loop you did not write is a loop you cannot get wrong.

| Function | Signature | Does |
| --- | --- | --- |
| `table.insert` | `(t, v)` or `(t, pos, v)` | Appends, or inserts and shifts the rest up |
| `table.remove` | `(t, pos?) -> any` | Takes the last element, or the one at `pos`, and closes the gap |
| `table.concat` | `(t, sep?) -> string` | Joins a list of strings and numbers; errors on anything else |
| `table.sort` | `(t, comparator?)` | Sorts a list **in place**, in byte order by default |
| `table.find` | `(t, v) -> number?` | The index of the first match in the list part, or `nil` |
| `table.move` | `(a1, f, e, t, a2?)` | Copies a range between lists without a loop |
| `table.clone` | `(t) -> table` | A new table with the same keys and values, one level deep |
| `table.freeze` | `(t) -> table` | Makes it read-only; a later write errors |
| `table.isfrozen` | `(t) -> boolean` | Whether it has been frozen |

Two of these have a sharp edge. **`table.sort` sorts in place** and returns nothing, so `local sorted = table.sort(names)` gives you `nil` and quietly rearranges `names`. And its default order is byte order, which puts every uppercase letter before every lowercase one: `{ "orange", "Apple", "banana", "Cherry" }` sorts to `Apple, Cherry, banana, orange`. Sorting case-insensitively means passing a comparator, which is a function — the next module's subject.

## Walking a table

**for k, v in t**

```luau
local mixed = { "first", "second", tag = "x", level = 3 }

for key, value in mixed do
	print(key, value)
end
```

Prints `1 first` and `2 second` first, then `level 3` and `tag x` in some order. This is **generalized iteration**: `for k, v in t` with the table itself, no wrapper function needed.

The order is only half-promised, and the half matters. The **list part is walked in index order**, first to last, before anything else. The rest — the string keys — come afterwards in an order the language does not define and you must not depend on. If you need named fields in a fixed order, keep a list of the key names beside the record and walk that.

Generalized iteration is a Luau addition. Older code writes `for i, v in ipairs(t)` to walk just the list part, stopping at the first `nil`, and `for k, v in pairs(t)` to visit every key. Both still work and both mean the same thing they always did; `for k, v in t` is simply shorter.

**The library at work**

```luau
local names = { "cy", "ada", "bob" }

table.insert(names, "dee")
table.sort(names)

print(table.concat(names, ", ")) -- ada, bob, cy, dee
print(table.find(names, "cy"))   -- 3
print(table.find(names, "zed"))  -- nil

local frozen = table.freeze(table.clone(names))
print(table.isfrozen(frozen), table.isfrozen(names))
```

The last line prints `true false`: `clone` made a separate table and `freeze` locked that one, leaving the original writable. Freezing the constant data at the top of a file is a cheap way to make an accidental write loud instead of mysterious.

**Key points**

- Nine table functions replace nine loops you would otherwise write by hand.
- `table.sort` and `table.insert` work **in place** and return nothing.
- The default sort is byte order, so uppercase sorts before lowercase.
- `for k, v in t` walks the list part in order, then the named keys in no defined order.
- `table.clone` copies one level; `table.freeze` makes an accidental write loud.
