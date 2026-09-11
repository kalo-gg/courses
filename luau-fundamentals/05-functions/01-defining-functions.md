# Defining and calling

*Declare with local function, call with parentheses, and pass functions around like any value.*

Luau Fundamentals · Functions & Scope · 9 min

---

A function is a piece of behaviour with a name on it. Everything you have written so far has been one long straight line of script; a function is how you say a thing once and use it in three places.

Luau has three ways to write one, and they build the same thing. The differences are about where the name lives, not about what the function is.

**Three ways to write the same function**

```luau
-- 1. A local function: the form you will write most
local function describe(name)
	return "the " .. name
end

-- 2. A function value assigned to a local
local describe2 = function(name)
	return "the " .. name
end

-- 3. A field on a table, which is how a module exposes one
local M = {}
function M.describe(name)
	return "the " .. name
end

print(describe("hall"), describe2("hall"), M.describe("hall"))
```

All three print `the hall`. Form 1 and form 2 differ in one small way: `local function f` declares the name *before* the body, so the body can call `f` — which is what makes recursion possible. Form 2's body cannot see `describe2` at all.

A function is a **value**. It has a `typeof` of `function`, it can be stored in a table, passed as an argument and returned from another function — you have already handed one to `table.sort` as a comparator without thinking about it.

The distinction that matters when you do this is between `f` and `f()`. `f` is the function itself; `f()` is the result of running it. Passing `f()` where a callback was wanted is the single most common mistake in this area, and it usually shows up as "my callback ran immediately and then nothing happened".

## Arguments are positional, and optional

Arguments are matched by **position**, and Luau does not check how many you passed.

- Pass fewer than the parameters and the missing ones are `nil`.
- Pass more and the extras are dropped.

Neither is an error. That is a design choice, not an oversight — it is what makes the `x or default` idiom work for optional parameters — but it does mean a mistyped or misordered argument list surfaces much later, as a `nil` somewhere in the body.

**Missing and extra arguments**

```luau
local function greet(name, level)
	print(name, level)
end

greet("Ada", 7)          -- Ada  7
greet("Ada")             -- Ada  nil
greet("Ada", 7, "extra") -- Ada  7, the third is dropped

-- The usual way to give a parameter a default:
local function greet2(name, level)
	level = level or 1
	print(name, level)
end

greet2("Bo") -- Bo  1
```

`level = level or 1` at the top of the body is Luau's whole story on default arguments: there is no `level = 1` in the parameter list. Remember the Module 2 warning — this idiom cannot express a default for a parameter whose legitimate value could be `false`.

**Key points**

- `local function f(…) … end` is the form to reach for; it lets the body see its own name, so recursion works.
- A function is a value: `typeof` says `function`, and it can be stored, passed and returned.
- `f` is the function and `f()` is its result; one pair of parentheses changes everything.
- Arguments are positional and unchecked: missing ones are `nil`, extras are dropped.
