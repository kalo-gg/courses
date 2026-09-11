# Any number of arguments

*The ... expression, select for counting, and table.pack when a nil has to be preserved.*

Luau Fundamentals · Functions & Scope · 10 min

---

`print` takes as many arguments as you give it. So can your functions: write `...` — three dots, spelled exactly that way — as the last parameter and it stands for "everything else that was passed".

`...` is not a table and not a variable. It is a **value list**, and it follows the same last-position rule as a multi-value call: put it last and all of it survives, put it anywhere else and only the first value does.

**... forwarded straight through**

```luau
local function log(...)
	print("[log]", ...)
end

log("a", "b", "c") -- [log]  a  b  c
log()              -- [log]

-- In the middle of a list, only the first value survives.
local function bad(...)
	print("[log]", ..., "end")
end

bad("a", "b", "c") -- [log]  a  end
```

The second function is the mistake to recognise: `...` was not last, so it contributed one value and the other two vanished silently. Forwarding arguments means putting `...` at the end.

To do anything but forward it, you need to know how many values it holds — and `#{ ... }` is the wrong way to ask. Collecting a value list into a table with a `nil` in it makes a table with a hole, and Module 4 told you what `#` does with one of those.

`select("#", ...)` is the right way. It counts the values that were actually passed, **including trailing `nil`s**, which is the entire reason the function exists. `select(n, ...)` then returns everything from the `n`th value onwards.

**Counting arguments**

```luau
local function count(...)
	print(select("#", ...), #{ ... })
end

count(1, 2, 3)       -- 3  3
count(1, nil, nil)   -- 3  1 : select is right, # is not

local function third(...)
	return (select(3, ...))
end

print(third("a", "b", "c", "d")) -- c
```

`select("#", 1, nil, nil)` is `3`, because three values were passed. `#{ 1, nil, nil }` answers whatever it likes — `1` in this sandbox — because the table has a hole. The parentheses around `select(3, ...)` keep only the third value instead of everything from there on.

## When you need them as a table

When you have to keep the arguments around rather than forward them straight on, `table.pack(...)` collects them into a table with an extra `n` field holding the true count. Read the values with the `n` rather than with `#` and the holes stop mattering.

`table.unpack(t)` goes the other way, turning a list back into a value list — and `table.unpack(t, 1, t.n)` is the form that survives the holes. Older code writes the global `unpack(t)`, which still works and is the deprecated spelling.

**Key points**

- `...` as the last parameter stands for every remaining argument, and follows the last-position rule.
- `select("#", ...)` counts them, trailing `nil`s included; `#{ ... }` does not.
- `select(n, ...)` returns everything from `n` onwards; wrap it in parentheses for just one.
- `table.pack(...)` keeps the true count in an `n` field; `table.unpack(t, 1, t.n)` puts it back.
