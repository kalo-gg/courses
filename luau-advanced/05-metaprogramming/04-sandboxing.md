# Handing over less than everything

*Frozen tables and proxies are what Luau left you for boundaries, and what neither of them can stop.*

Luau Advanced · Metamethods in Full · 10 min

---

Luau's designers removed a lot on purpose. There is no `io`, no `dofile`, no `loadstring`, and `getfenv`/`setfenv` are deprecated because they defeat the compiler's assumptions about globals. What is left for putting a boundary between your code and someone else's is exactly two things: **a frozen table**, and **a proxy**.

That is less than it sounds like, and enough for most real boundaries.

**A frozen API surface**

```luau
local api = table.freeze({
	version = "1.0",
	add = function(a: number, b: number)
		return a + b
	end,
})

print(api.add(2, 3), table.isfrozen(api))

local ok, err = pcall(function()
	(api :: any).add = nil
end)
print(ok, string.match(tostring(err), "attempt.*"))
```

Prints `5	true`, then `false	attempt to modify a readonly table`. Nobody can swap `add` for something of their own, which is the guarantee callers of `api` get to rely on.

Freezing stops replacement. A proxy adds the other half: **hiding**. An empty table with `__index` can serve computed values, redact some of them, and refuse writes with a message of your own — and `__metatable` stops the caller from reading or replacing the metatable to get around any of it.

**A proxy that hides and locks**

```luau
local secrets = { key = "abc", public = "ok" }

local view = setmetatable({}, {
	__index = function(_, k)
		return k == "key" and "<hidden>" or secrets[k]
	end,
	__newindex = function()
		error("read only")
	end,
	__metatable = "locked", -- getmetatable returns this; setmetatable refuses
})

print(view.key, view.public, getmetatable(view))
local ok, err = pcall(function()
	(view :: any).x = 1
end)
print(ok, string.match(tostring(err), "read only"))
```

Prints `<hidden>	ok	locked`, then `false	read only`. `setmetatable(view, {})` raises `cannot change a protected metatable`.

## What a sandbox cannot stop

Neither tool stops code from **not finishing**. `while true do end` uses no globals, writes to nothing and never returns, and no metatable can help.

That problem is solved one level up, by the host. This sandbox uses an **interrupt limit**: a budget of VM instructions, after which the run is stopped. Roblox has its own equivalent. The lesson to take from that is where each defence lives: a frozen table and a proxy control *what code can reach*, and only the host can control *how long it runs* or how much memory it takes.

> **This page runs inside one**
>
> The Kalo runtime is exactly this idea. Your code gets core Luau — `string`, `table`, `math`, `coroutine`, `utf8`, `bit32`, `buffer`, `vector`, and `gcinfo` — and nothing else: no `game`, no `task`, and no `os`, `debug` or `io`, which is why this course counts operations rather than timing them. Around that sit the host's limits: an instruction budget, a memory cap and a byte limit on output. Everything in this lesson is being done to you as you read it.

**Key points**

- Luau removed `io`, `dofile` and `loadstring` and deprecated `getfenv`/`setfenv` for sandboxing reasons.
- A frozen table stops replacement; a proxy adds hiding, computed values and refused writes.
- `__metatable` blocks reading or replacing the metatable that enforces all of it.
- Neither can stop an infinite loop or runaway memory — those are the host's limits, and this page runs under exactly such a set.
