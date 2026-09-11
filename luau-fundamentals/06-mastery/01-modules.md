# Modules and require

*A file that returns a table becomes a module, and require gives every caller the same copy.*

Luau Fundamentals · Modules & Mistakes · 10 min

---

There is no `module` keyword in Luau, and no `export`. A **module** is just a file that ends with a `return`, and whatever it returns is what the caller receives.

By convention that is a table of functions, because a table can hold as many exports as you need and can grow one later without changing anything at the call site.

**greeting.luau — a module is a file that returns something**

```luau
local greeting = {}

function greeting.hello(name)
	return "Hello, " .. name
end

function greeting.bye(name)
	return "Bye, " .. name
end

return greeting
```

The local table is built, filled and returned. Everything not put into that table — a helper function, a constant, a cache — stays private to the file, because a local is only visible inside the block it was declared in.

**main.luau — requiring it**

```luau
local greeting = require("./greeting")

print(greeting.hello("Ada")) -- Hello, Ada
print(greeting.bye("Ada"))   -- Bye, Ada
```

This pair of files does not run in a lesson exercise: the sandbox only registers modules for a **project**, so a `require` typed into the editor here fails with `module not found`. The exercise below builds the table without the `require`, and the project at the end of this module proves the whole shape.

A module's body runs **once**. The first `require` executes the file top to bottom and remembers what it returned; every later `require` of the same module gets that same value back without running anything again.

That is usually exactly what you want — you do not want three copies of a config table — and it is occasionally a trap. A module that holds state shares that state with every caller in the program. If two systems both `require("./inventory")`, they are looking at one inventory, not two.

## Three ways to say where

| Where you are | How you write it | What the argument is |
| --- | --- | --- |
| Roblox | `require(script.Parent.greeting)` | An Instance in the DataModel, found by navigating from `script` |
| Standalone Luau, Lune | `require("./greeting")` | A path relative to the requiring file; the `./` prefix is required |
| This sandbox, inside a project | `require("./greeting")` | The project's own files, registered by path — the same spelling as standalone |

The courses here write the `./` form, because it is what real standalone Luau requires and what the sandbox accepts.

> **`require` is not part of the language**
>
> The Luau VM defines no `require` at all. Roblox supplies one that takes an Instance, Lune supplies one that takes a path, and this sandbox supplies one that knows the project's files. That is why the path syntax differs between them, why a Roblox snippet's `require(script.Parent.X)` cannot work outside Studio, and why a library that wants to run in both places keeps its requires at the very top where they are easy to swap.

Two habits make modules pleasant to use. **Put every `require` at the top of the file**, so a reader can see the whole dependency list in one glance and a circular dependency is obvious rather than mysterious. And **return one table, built as a local**: `local M = {}` … `return M` is the shape everyone recognises, and it lets the module's own functions call each other through `M`.

**Key points**

- A module is a file that returns something; by convention, one table of functions.
- Anything not put in that table is private, because a file-level `local` is invisible outside the file.
- A module body runs once and its return value is cached for every later `require`.
- `require` comes from the host, not the language, which is why the path syntax differs everywhere.
