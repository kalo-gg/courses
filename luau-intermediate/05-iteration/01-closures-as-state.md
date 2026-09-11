# Functions that remember

*A closure captures the variable, which makes it the smallest object Luau has.*

Luau Intermediate · Iterators & Closures · 9 min

---

You have made a closure before: a function returned from another function, still able to see the locals it was born among. What is worth being precise about is *what* it captures, because the answer decides whether two closures made together share anything.

A closure captures the **variable**, not a copy of its value. Two closures made in the same scope see the same variable and each other's writes; two closures made by separate calls to the same factory see different variables entirely.

**Each loop iteration captures its own variable**

```luau
local handlers = {}

for _, name in { "lamp", "rope" } do
	handlers[name] = function()
		return "handling " .. name
	end
end

print(handlers.lamp(), handlers.rope())
```

Prints `handling lamp  handling rope`. Every pass of the loop creates a fresh `name`, so the two closures captured two different variables. Languages where this prints the last value twice are the reason it is worth checking.

**Two closures over one variable share it**

```luau
local function makeGate()
	local open = false

	local function toggle()
		open = not open
	end

	local function isOpen()
		return open
	end

	return toggle, isOpen
end

local toggle, isOpen = makeGate()
print(isOpen())
toggle()
print(isOpen())
```

Prints `false`, then `true`. `toggle` and `isOpen` were made in the same call, so they share one `open`. A second call to `makeGate` produces a second, independent pair.

That is the whole model, and it is enough to build objects out of. `makeGate` is a thing with private state and two operations, and nothing outside can read `open` except through `isOpen`. Luau has no `private` keyword; this is what it has instead.

The cost is that you cannot inspect it, extend it, or give it a metatable. Which is exactly the trade-off the next section is about.

## A closure against a table

|  | A closure | A table with a metatable |
| --- | --- | --- |
| State | Private by construction — nothing can reach it | Public fields; privacy is a convention |
| Behaviours | One, or a small fixed set returned together | As many methods as you like, added later |
| Inspection | None: you cannot print it or look inside | `print`, `__tostring`, iteration over fields |
| Extension | Not possible | Inheritance, or a second metatable |
| Cost | One function object per behaviour, per instance | One shared method table for every instance |

A closure is the smallest object Luau has. Reach for a table the moment you want a second method, or want to see inside.

The row that decides it most often is inspection. A closure cannot be printed, looped over or given a `__tostring`, so the moment you want to see what one is holding you have to add a function that tells you — and by then you have written a very small table with worse ergonomics.

The next three lessons are closures doing the job they are best at: iterators, which are one behaviour with private position and nothing worth looking inside.

**Key points**

- A closure captures the **variable**, not a copy of its value.
- Closures made in the same call share their captured locals; closures from separate calls do not.
- Each pass of a Luau loop creates fresh loop variables, so closures made in a loop capture different ones.
- A closure is the smallest object there is: private state, one behaviour, nothing to inspect.
- Reach for a table once you want several methods, printing, iteration or extension.
