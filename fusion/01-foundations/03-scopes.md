# Scopes own what you create

*A scope is an array of things to destroy. Everything you make goes in one, and it is cleaned up in reverse.*

Fusion · Fusion Foundations · 11 min

---

Here is the idea in one sentence, and the rest of the lesson is detail: **a scope is an array of destructors, and every Fusion object adds itself to one when it is created.** Destroying the scope destroys everything in it, so you never keep a list of things to tidy up by hand.

**A scope really is an array**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)

local thing = scope:Value("i am a thing")

print(#scope) --> 1
print(scope[1]) --> function: 0x… (the destructor for the value above)
```

Nothing magic is happening. The object put its own cleanup function into the array you passed it.

## Why `scoped(Fusion)` and not `{}`

A scope can be a plain table, and `Fusion.Value(myTable, 0)` works. But `Fusion.scoped(Fusion)` merges Fusion's constructors into the scope, so you can write `scope:Value(0)` instead of naming the scope twice.

The method form is the shape the documentation uses throughout, and it is what this course uses.

**The same code, three ways**

```luau
-- Explicit, and you name the scope twice
local scope = {}
local a = Fusion.Value(scope, 0)

-- scoped() merges the constructors in
local scope = Fusion.scoped(Fusion)
local b = scope:Value(0)

-- Your own helpers can ride along too
local scope = Fusion.scoped(Fusion, myComponents)
local c = scope:Button { Text = "Buy" }
```

## Cleaning up

**doCleanup destroys in reverse**

```luau
local scope = Fusion.scoped(Fusion)

local first = scope:Value(1)
local second = scope:Value(2)
local gui = scope:New "ScreenGui" { Parent = playerGui }

Fusion.doCleanup(scope) -- gui, then second, then first
```

The array is walked backwards, so an object is always destroyed before whatever it was built from.

`doCleanup` is not limited to Fusion objects. Per its own documentation it destroys an `Instance`, disconnects an `RBXScriptConnection`, calls a function, calls `destroy` or `Destroy` on a table that has one, and walks into arrays of any of those. That means your own objects can live in a Fusion scope as long as they offer one of those shapes.

> **A cleaned-up scope is poisoned**
>
> `doCleanup` does not just empty the scope — it **poisons** it, replacing its metamethods so that any later read or write errors with *"Attempted to use a scope after it's been destroyed"*. That is deliberate, because Fusion reserves the right to recycle the table. So a scope is single-use: clean it up, then make a fresh one with `scoped` rather than reaching for the old variable.

**Key points**

- A scope is an array of destructors; every constructor's first argument is the scope that will own the result.
- `scoped(Fusion)` merges the constructors in, so `scope:Value(0)` works; a plain table works too, with `Fusion.Value(scope, 0)`.
- `doCleanup` walks the array backwards: instances destroyed, connections disconnected, functions run, `destroy`/`Destroy` methods called.
- A cleaned-up scope is poisoned; never reuse it.
