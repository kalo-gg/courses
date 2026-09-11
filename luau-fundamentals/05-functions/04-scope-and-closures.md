# Scope and closures

*Where a local is visible, how long it lives, and what a function captures when it is created.*

Luau Fundamentals · Functions & Scope · 10 min

---

Scope in Luau is **lexical**: a `local` is visible from its declaration to the end of the block it was declared in, and nowhere else. A block is a function body, a loop body, an `if` branch, or a bare `do … end`.

That means the answer to "can this line see that variable?" is always visible in the source. You never have to know what called what.

**A local ends with its block**

```luau
do
	local inner = "only in here"
	print(inner) -- only in here
end

print(inner) -- nil: the local is gone, and this reads a global that was never set

local shadowed = "outer"
do
	local shadowed = "inner" -- a different variable with the same name
	print(shadowed)          -- inner
end
print(shadowed)              -- outer
```

Reading a local that has gone out of scope is not an error: the name falls through to the global environment, which does not have it either, so you get `nil`. That quiet `nil` is the same one from Module 1, arriving by a third route.

## A function remembers where it was made

A function body can read the locals that surrounded it when it was **created**, and it keeps them alive for as long as the function exists. A function that does this is called a **closure**, and in Luau every function is one.

The important detail: a closure captures the **variable**, not a snapshot of its value. If the variable changes, the closure sees the new value. And if two closures capture the same variable, they share it.

**A counter**

```luau
local function makeCounter(start)
	local n = start
	return function()
		n += 1
		return n
	end
end

local a = makeCounter(0)
local b = makeCounter(10)

print(a(), a(), b()) -- 1  2  11
```

Each call to `makeCounter` creates a fresh `n`, so `a` and `b` count independently. `n` is not visible to anything outside — the only way to reach it is by calling the function that captured it, which is private state without a single keyword for it.

Two practical consequences.

First, **a captured variable does not go away when its block does.** The `n` inside `makeCounter` outlives the call that created it, because a live function still refers to it. That is not a leak; it is how the counter works.

Second, **shadowing is allowed and is usually a mistake.** Declaring a `local` with a name that already exists in an outer scope creates a second, unrelated variable. It compiles, it runs, and it silently ignores every write you thought you were making to the outer one. A linter such as Selene will point at it.

**Key points**

- Scope is lexical: a `local` lives from its declaration to the end of its block.
- Every function captures the surrounding locals it mentions, and keeps them alive.
- A closure captures the **variable**, not a snapshot, so two closures over one local share it.
- The numeric `for` variable is fresh each pass, so closures made in a loop capture different values.
- Shadowing compiles and runs, and is almost always a mistake.
