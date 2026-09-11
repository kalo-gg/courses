# Narrowing a value

*Guards teach the checker what a value is, and a call between the guard and the use can unteach it.*

Luau Advanced · The Type System, Deeply · 10 min

---

Luau Intermediate showed you `string | number` and the `if typeof(x) == "string"` that goes with it. What it did not say is that the narrowing is a piece of **flow analysis**: the checker walks your control flow and remembers what each branch has proved. That has consequences, and the consequences are what this lesson is about.

None of it changes what the program does. Types are erased before the VM sees a thing — the code below runs identically with every annotation deleted.

**A guard narrows inside its branch**

```luau
--!strict
local function describe(x: string | number): string
	if typeof(x) == "string" then
		return "text of " .. #x .. " characters" -- x is a string here
	end
	return "number " .. x * 2 -- and a number here, because the branch returned
end

print(describe("hey"))
print(describe(4))
```

Prints `text of 3 characters` then `number 8`. `#x` would be an error on a number and `x * 2` on a string; each is fine because the checker knows which branch it is in.

Notice the second narrowing. The `if` branch **returns**, so everything after it is unreachable when `x` was a string — the checker takes that into account and narrows the rest of the function to `number` without a second guard.

`assert` does the same thing for the whole remaining scope. `assert(v ~= nil, "…")` is not just a run-time check; the code after it only runs when the condition held, so the checker narrows from there on.

## What breaks a refinement

A refinement is a claim about a *place*, and the checker throws it away as soon as it cannot prove the place still holds what it did. Two things do that:

- **A call.** Any function in scope might write to the variable, so the checker discards refinements on anything it does not fully control across a call.
- **A reassignment.** Writing to the variable obviously invalidates what was known about it.

A refinement on a *table field* is the fragile one: `t.x` can be written by anything holding `t`. Assign it to a local first and narrow the local, which nothing else can touch.

**A refinement lost**

```luau
--!strict
local current: string? = "ready"

local function clear()
	current = nil
end

if current ~= nil then
	clear() -- the checker gives up here: `clear` can write to `current`
	print(current) -- `string?` again, and at run time this really is nil
end
```

Prints `nil`. The refinement was not merely forgotten by a pedantic checker — the call genuinely invalidated it, which is the point.

**Key points**

- Narrowing is flow analysis: the checker remembers what each branch has proved.
- An early `return` narrows everything after it; `assert` narrows the rest of the scope.
- A call or a reassignment invalidates a refinement, because neither can be proved harmless.
- Refine a private local rather than a table field — nothing else can write to it.
