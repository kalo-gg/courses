# Designing the thing someone else calls

*Fewer arguments, obvious names, no surprising mutation, and a return that says what happened.*

Luau Intermediate · Structuring a Codebase · 9 min

---

Everything in this module has been about the boundary. This lesson is about what the boundary *looks like* to the person on the other side of it — who is, in six months, you.

Four rules cover most of it, and each one exists because of a specific way an API gets misused.

| Rule | The misuse it prevents |
| --- | --- |
| Take few arguments; past three, take an options table | `add(store, "lamp", 2, nil, true)` — nobody can read the call site, and two of the arguments are in the wrong order half the time |
| Name the function after what it does to what | `process(store)` gets called in the wrong place because nobody knows what it processes |
| Mutate or return, not both | `local s = sort(store)` looks pure, edits the caller's table, and now two names point at one thing |
| Return something that says what happened | A function that silently does nothing on failure produces a bug report about the *next* function |

None of these is about types. A perfectly typed API can break all four.

**The same API, twice**

```luau
--!strict
type Store = { [string]: number }

-- Hard to use: five positional arguments, and it mutates and returns.
local function addItem(store: Store, name: string, qty: number, force: boolean, quiet: boolean): Store
	if force or store[name] == nil then
		store[name] = qty
	end
	return store
end

-- Easier: one options table, one job, and a result that says what happened.
type AddOptions = { name: string, qty: number, replace: boolean? }

local function add(store: Store, opts: AddOptions): (boolean, string?)
	if store[opts.name] ~= nil and not opts.replace then
		return false, "duplicate " .. opts.name
	end
	store[opts.name] = opts.qty
	return true, nil
end

local store: Store = {}
print(add(store, { name = "lamp", qty = 2 }))
print(add(store, { name = "lamp", qty = 5 }))
```

Prints `true  nil`, then `false  duplicate lamp`. The options table also makes the optional argument optional in fact rather than in spirit — nobody has to pass `nil, false` to reach the one they wanted.

The mutation rule is the one worth spelling out. A function that edits its argument **and** returns it is the hardest kind to reason about, because the return value is a lie by omission: it looks like a new thing and it is the old one.

So pick. `table.sort` mutates and returns nothing, which is honest. A `sorted(list)` that builds a copy and returns it is honest too. `sort(list)` returning the same list is what produces the bug where two variables mysteriously change together.

## Naming is the cheapest documentation

A name is read every time the function is called and written once. That ratio is why it is worth more thought than it usually gets.

The reliable pattern is **verb plus object**: `addItem`, `findSlot`, `clearBag`. A bare verb (`process`, `handle`, `update`) says nothing about what it operates on, and a bare noun (`items()`, `slot()`) hides whether anything happens. And say what the function *does*, not how — `sorted` rather than `quickSort`, because the how is allowed to change.

**Two names for the same function**

```luau
--!strict
-- What does it process, and what comes back?
local function process(t: { number }): { number }
	local out: { number } = {}
	for _, n in t do
		if n > 0 then
			table.insert(out, n)
		end
	end
	return out
end

-- Same body, one better name. Now the call site explains itself.
local function positiveOnly(numbers: { number }): { number }
	local out: { number } = {}
	for _, n in numbers do
		if n > 0 then
			table.insert(out, n)
		end
	end
	return out
end

print(#process({ 1, -2, 3 }), #positiveOnly({ 1, -2, 3 }))
```

Prints `2  2` — identical behaviour. Nothing about the implementation changed; the call site went from unreadable to obvious.

**Key points**

- Past three arguments, take an options table — it names the arguments at the call site and makes optionals genuinely optional.
- Name functions verb-plus-object, after what they do rather than how they do it.
- Mutate or return, never both; a function that edits and returns its argument is the hardest to reason about.
- Return something that says what happened, so a failure surfaces here rather than three functions later.
- None of these is a typing problem: a fully annotated API can still break all four.
