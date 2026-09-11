# Failures a caller can handle

*error is for a bug, a returned ok and message is for a failure the caller was always going to hit.*

Luau Intermediate · Structuring a Codebase · 10 min

---

There are two kinds of failure and they want different mechanisms. A **bug** — an index that cannot happen, a state that should be unreachable — should stop the program loudly, and `error` does that. An **expected failure** — the inventory is full, the item is not there, the input was blank — is not a bug at all: it is one of the outcomes the function was written to have.

Spending `pcall` on the second kind is the mistake. It turns a normal outcome into an exception and back again, and it hides the real bugs that happen to fly past at the same time.

| Style | Use it for | The caller writes |
| --- | --- | --- |
| `error(msg)` | Programmer mistakes, unreachable states, broken invariants | Nothing — it should crash, and the traceback is the point |
| `return false, msg` | Failures the caller was always going to hit: full, missing, invalid | `local ok, err = f()` and an `if` |
| `return Result<T>` | The same, when the success case carries a value worth naming | `if r.ok then … else … end` |

A `pcall` around your own module's routine failure is a sign the module chose the wrong style, not a sign the caller is being careful.

**The same function, both ways**

```luau
--!strict
type Store = { [string]: number }

-- Throws: every caller has to wrap the call to survive it.
local function takeOrThrow(store: Store, name: string, qty: number): number
	local have = store[name] or 0
	if have < qty then
		error("not enough " .. name)
	end
	store[name] = have - qty
	return store[name]
end

-- Returns: the failure is part of the signature.
local function take(store: Store, name: string, qty: number): (boolean, string?)
	local have = store[name] or 0
	if have < qty then
		return false, "not enough " .. name
	end
	store[name] = have - qty
	return true, nil
end

print(take({ lamp = 3 }, "lamp", 5))
print(take({ lamp = 3 }, "lamp", 2))
```

Prints `false  not enough lamp`, then `true  nil`. Note also what `pcall(takeOrThrow, …)` gives back: `false  main.luau:8: not enough lamp` — `error` prefixes the position, so a thrown message is not a string you want to show a player.

The returning version costs one extra line at each call site and buys two things. The signature now documents the failure, so nobody has to read the body to find out it can go wrong. And `(boolean, string?)` is a type Typing Data already gave you the vocabulary for — the checker will make callers deal with the `string?`.

An honest rule of thumb: if you can imagine writing a test that expects the failure, it is a returned failure. If the only sensible test is "this must never happen", it is an `error`.

## Typing the result

**A tagged `Result`**

```luau
--!strict
type Result<T> = { ok: true, value: T } | { ok: false, err: string }

local function find(store: { [string]: number }, name: string): Result<number>
	local qty = store[name]
	if qty == nil then
		return { ok = false, err = "no such item: " .. name }
	end
	return { ok = true, value = qty }
end

local r = find({ lamp = 3 }, "rope")
if r.ok then
	print("found", r.value)
else
	print("missing", r.err)
end
```

Prints `missing  no such item: rope`. This is the tagged union from Typing Data with a boolean tag: `if r.ok then` narrows the type, so `r.value` is legal in one branch and `r.err` in the other, and neither is legal in both.

Two shapes, one idea. `(boolean, string?)` is lighter and reads well when the success case carries nothing interesting. `Result<T>` is worth the extra type when success has a value, because the checker then refuses to let anyone read it without checking.

Pick one per module and stay with it. A codebase where half the functions return pairs and half return tagged tables makes every call site a small decision.

**Key points**

- `error` is for bugs and unreachable states; a returned failure is for outcomes a correct program hits.
- `pcall` around your own module's routine failure means the module chose the wrong style.
- `(boolean, string?)` documents the failure in the signature and makes the checker enforce handling it.
- `Result<T>` is worth the extra type when success carries a value, because the tag gates the read.
- Handle the failure first and return early; keep the success path unindented.
