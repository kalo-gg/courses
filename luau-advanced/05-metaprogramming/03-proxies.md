# Intercepting everything

*An empty table plus __index and __newindex sees every access, and charges a function call for each.*

Luau Advanced · Metamethods in Full · 11 min

---

Luau Intermediate used `__index` to fall back to a class table, and `__newindex` to refuse writes. A **proxy** uses both at once, over storage the caller cannot reach, so every single access goes through code you wrote.

The whole trick is one requirement: the proxy table has to stay **permanently empty**. `__index` fires only for keys the table does not have, and `__newindex` fires only for keys it does not have. Put anything in the proxy and that key is invisible from then on.

**An empty table over a hidden store**

```luau
local store = { hp = 10 }
local reads, writes = 0, 0

local proxy = setmetatable({}, {
	__index = function(_, key)
		reads += 1
		return store[key]
	end,
	__newindex = function(_, key, value)
		writes += 1
		store[key] = value -- into `store`, never into the proxy
	end,
})

proxy.hp = 12
print(proxy.hp, reads, writes)
```

Prints `12	1	1`. The write went to `store`, so the proxy is still empty and the next read still fires `__index`.

That last line in `__newindex` is the one to get right. Writing `rawset(proxy, key, value)` would also work — once. From then on the proxy owns that key, both hooks skip it, and the interception you built quietly stops applying to exactly the fields people use most.

## What it costs

A proxy turns every field access into a **function call**. Not a slightly slower lookup — a call, with the argument passing and the return that go with it. Module 1's price list puts a table field near the cheap end and a call several rows above it, and a proxy moves every access from the first to the second.

That makes it the right tool for a boundary or a debug build, and the wrong tool inside a loop that runs per frame. The good news is that the swap is invisible to callers: a proxy and the table it wraps are used identically, so you can install one while investigating and take it out afterwards.

**A logging proxy**

```luau
local log = {}
local target = { name = "core" }

local view = setmetatable({}, {
	__index = function(_, key)
		table.insert(log, "read " .. tostring(key))
		return target[key]
	end,
})

print(view.name, view.missing)
print(table.concat(log, " | "))
```

Prints `core	nil`, then `read name | read missing`. Even the miss is recorded, which is usually the access you were looking for.

**Key points**

- A proxy is an empty table whose `__index` and `__newindex` reach a hidden store.
- Both hooks fire only for absent keys, so the proxy must never store anything.
- `rawget` and `rawset` are the deliberate way past the hooks, and the accidental way to disable them.
- Every access becomes a function call: use a proxy on a boundary or in a debug build, never in a hot loop.
