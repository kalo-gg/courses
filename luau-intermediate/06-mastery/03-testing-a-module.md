# Testing what you wrote

*A test is a function that asserts; a framework only organises them, and assert alone gets you started.*

Luau Intermediate · Structuring a Codebase · 10 min

---

A test is not a special kind of code. It is a call to your function and an `assert` about what came back. You can have three of them at the bottom of a file in the next minute, and they will catch more than the framework you keep meaning to set up.

`assert(value, message)` throws when the value is falsy and does nothing otherwise, so a file that runs to the end in silence is a file whose tests all passed.

**Three asserts, run at the bottom of the file**

```luau
local function add(store: { [string]: number }, name: string, qty: number): (boolean, string?)
	if store[name] ~= nil then
		return false, "duplicate " .. name
	end
	store[name] = qty
	return true, nil
end

local s: { [string]: number } = {}
assert(add(s, "lamp", 2) == true, "a new name is accepted")
assert(s.lamp == 2, "the quantity is stored")
assert(add(s, "lamp", 1) == false, "a duplicate name is refused")
print("all tests passed")
```

Prints `all tests passed`. The three asserts cover the accepted case, the effect on the store, and the refusal — and the third is the one that would have caught the bug of forgetting the duplicate check.

What to test first is not "everything". It is the **boundary cases the type system cannot express**: the empty list, the duplicate, the container that is exactly full, the quantity of zero. Types will tell you a `number` arrived; only a test will tell you what happens when that number is the capacity.

The happy path usually tests itself the moment you run the program. The edges do not.

## When to reach for a framework

Asserts at the bottom of a file stop scaling for four reasons, and none of them is about assertions. You want **names** so a failure says which case broke; **grouping** so related cases sit together; **a report** so you see all the failures rather than just the first; and a way to run tests **only in test builds** rather than every time the module loads.

That is what a runner gives you. Kalo has a whole course on **Lest**, Luau's, and this lesson deliberately does not re-teach it — nothing here needs it installed. Reach for it when the four reasons above start to bite, which is usually around the second module you write tests for.

> **A test that cannot fail is worse than no test**
>
> `local ok = pcall(f) assert(ok)` only checks that `f` did not throw. If `f` returns the wrong answer, the test is green. Assert on the **value**: `local accepted, err = add({}, "lamp", 2)` then `assert(accepted == true and err == nil)`. A suite full of no-throw checks is a suite that will keep reporting success through the whole of the bug you are hunting.

**The same case, badly and well**

```luau
-- Cannot fail: pcall swallows the result and nothing is asserted about it.
local ok = pcall(function()
	return add({}, "lamp", 2)
end)
assert(ok, "add did not throw")

-- Can fail: it asserts on what came back.
local accepted, err = add({}, "lamp", 2)
assert(accepted == true, "add should accept a new name")
assert(err == nil, "a successful add reports no error")
```

Both run clean today. Break `add` so it returns `false, "nope"` and only the second pair notices.

**Key points**

- A test is a call plus an `assert` on what came back; three at the bottom of a file beat a framework you have not set up.
- Test the **boundaries the types cannot express**: empty, duplicate, exactly full, zero.
- A test that only checks "it did not throw" cannot fail and makes the suite lie.
- Reach for a runner when you need names, grouping, a report, or tests that do not run on import.
- Kalo's Lest course is that runner; nothing in this course requires it.
