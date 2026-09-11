# Reads that miss and writes that are new

*One handles reads that miss, the other handles writes to keys that are absent, and neither is symmetric.*

Luau Intermediate · Metatables · 10 min

---

`__index` and `__newindex` look like a pair and behave like one only halfway. Both fire on a key the table does not have. But a read that misses is common and a write that is new is common too, so the second one fires far more often than people expect — and then stops firing for that key forever.

That is the asymmetry this lesson is about, and it is the source of most metatable bugs.

**`__index` as a table, and as a function**

```luau
local base = { greeting = "hello", volume = 5 }

local asTable = setmetatable({}, { __index = base })

local asFunction = setmetatable({}, {
	__index = function(_, key)
		return "no " .. key
	end,
})

print(asTable.greeting, asTable.missing)
print(asFunction.greeting)
```

Prints `hello  nil`, then `no greeting`. The table form chains: a miss on `asTable` becomes a lookup in `base`, and a miss there is a plain `nil`. The function form answers everything, which is powerful and easy to overdo.

## Writes are the other half

**`__newindex` and `rawset`**

```luau
local log = {}

local t = setmetatable({ x = 1 }, {
	__newindex = function(tbl, key, value)
		table.insert(log, key)
		rawset(tbl, key, value)
	end,
})

t.x = 2   -- x is already there: __newindex does NOT fire
t.y = 3   -- y is absent: it fires

print(t.x, t.y, table.concat(log, ","))
```

Prints `2  3  y`. Only `y` is logged. And note the `rawset`: a `__newindex` that assigns with `tbl[key] = value` would call itself forever, so the raw write is not optional.

Two rules fall out of that block. **`__newindex` fires only for keys that are not already present** — write to `x` a hundred times and the hook runs zero times. And **inside a `__newindex` you must use `rawset`**, because an ordinary assignment re-enters the hook.

If you want every write intercepted, the usual trick is to keep the table you hand out empty and store the real data somewhere else, so every key is always absent. The read-only view in this lesson's exercise is the simplest version of that.

`rawget` and `rawset` are the escape hatches for both hooks, and they are the right tool in exactly two places: inside a hook, to avoid recursion, and in code that genuinely wants to know what is stored rather than what is answered.

Everywhere else, use the ordinary syntax. Raw access sprinkled through normal code is how a metatable stops meaning anything.

**Key points**

- `__index` handles reads that miss; `__newindex` handles writes to keys that are **not already present**.
- Write to an existing key and `__newindex` never runs — the pair is not symmetric.
- Inside `__newindex`, assign with `rawset` or the hook calls itself forever.
- To intercept every write, keep the exposed table empty and store the data elsewhere.
- `rawget` and `rawset` belong inside hooks and in code that wants what is stored, not what is answered.
