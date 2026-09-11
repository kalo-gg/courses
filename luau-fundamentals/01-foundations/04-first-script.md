# Your first script

*Put values, variables and printing together into a script that greets a named player.*

Luau Fundamentals · Luau Foundations · 10 min

---

Everything so far, in one program. A script is a file that runs top to bottom: each line is executed in order, and a name has to exist before it can be read.

The one new piece here is `..`, the concatenation operator. It joins two strings, and it will convert a number for you — but nothing else, which is why concatenating a `nil` was an error back in Values and variables.

**greet.luau**

```luau
local name = "Ada"
local level = 7

local greeting = "Welcome back, " .. name .. "!"

print(greeting)
print("You are level " .. level .. ".")
```

Prints `Welcome back, Ada!` and then `You are level 7.`. The number `7` is converted to text by `..`; a `nil` or a table in the same position would raise `attempt to concatenate`.

## Comments

`--` starts a comment that runs to the end of the line. `--[[ … ]]` is a block comment that can span as many lines as you like. Luau ignores both entirely.

A comment is worth its line when it says something the code cannot: why a number is what it is, why a check is there, what a caller is expected to have done first. A comment that restates the line below it goes stale the first time somebody edits that line and forgets it.

**The same script, commented**

```luau
--[[
	greet.luau — the welcome line shown when a session starts.
	Levels come from the save file, so this file only formats them.
]]

local name = "Ada"
local level = 7 -- 7 is the first level that unlocks the market

local greeting = "Welcome back, " .. name .. "!"

print(greeting)
print("You are level " .. level .. ".")
```

Same output. The block comment says what the file is for; the line comment says why `7` and not some other number. Neither one says `-- set level to 7`.

Order matters. Luau reads a file from the top, so a `local` has to be declared before the line that reads it. Read a name too early and you do not get an error about the order — you get `nil`, because an undeclared name is a global and an unset global is `nil`. That is the same quiet failure from the last lesson, arriving by a different route.

**Key points**

- A script runs top to bottom, so every name must be declared before it is read.
- `..` joins strings and converts numbers, but refuses `nil` and tables.
- `--` comments to the end of the line, `--[[ … ]]` across lines.
- Write comments that say why, not comments that restate the line under them.
