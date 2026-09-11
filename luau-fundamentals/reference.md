# Luau Fundamentals reference

*Luau Fundamentals 0.733, released August 7, 2026. Checked on September 9, 2026.*

---

## Values and variables

A value is a thing, a variable is a name for one, and `typeof` is how you ask which kind you are holding.

### `local`

```luau
local name = value
local a, b = 1, 2
```

Declares a variable visible from the declaration to the end of the enclosing block. A name written without `local` is a global instead: it outlives its block, is a table lookup rather than a register, and leaks between files.

> Always write `local`. It is the single cheapest habit in the language, and the one that keeps a typo from becoming a shared variable.

### `local const`

```luau
local const name = value
```

A local the compiler is told will never be reassigned. Useful for the fixed numbers and strings at the top of a file.

> New syntax, and accepted by the 0.733 sandbox. Older Luau (and every Lua 5.1 tutorial) has no such form, and Lua 5.4's spelling is `local name <const> = value`, which does not parse here.

### `typeof`

```luau
typeof(v) -> string
```

The runtime kind of a value: `number`, `string`, `boolean`, `nil`, `table`, `function`, `thread`, `userdata`, `vector` or `buffer`.

> Prefer it over `type`. Inside Roblox `typeof` also names engine types (`Vector3`, `Instance`), where `type` answers `userdata` for all of them.

### `type`

```luau
type(v) -> string
```

Lua's original kind test. Identical to `typeof` for the plain values this course uses.

> Kept for compatibility with Lua 5.1 code. New Luau writes `typeof`.

### `tonumber`

```luau
tonumber(v, base?) -> number?
```

Converts a string to a number, or returns `nil` when it is not one. `tonumber("0x10")` is 16 and `tonumber("101", 2)` is 5.

> It returns `nil` rather than erroring, so a failed conversion travels quietly until something tries to use it — check the result at the point of conversion.

### `tostring`

```luau
tostring(v) -> string
```

The printable form of any value. `tostring(nil)` is `"nil"`, and a table gives its address.

> String interpolation (`` `{v}` ``) calls this for you; `..` does not, which is why concatenating a `nil` errors while interpolating one prints `nil`.

## Operators

Lua 5.1's operators plus the four things Luau added: floor division, compound assignment, `continue` and interpolation.

### `..`

```luau
a .. b -> string
```

Concatenation. Both sides must be a string or a number; anything else errors.

> Right-associative, which is why `"Hi " .. who.title .. " " .. who.name` reports `attempt to concatenate nil with string` — the failing pair is the inner one, not the one you wrote first.

### `//`

```luau
a // b -> number
```

Floor division: divides and rounds down. `7 // 2` is `3`.

> It rounds down, not towards zero, so `-7 // 2` is `-4`. Added by Luau; Lua 5.1 has no `//`.

### `%`

```luau
a % b -> number
```

Remainder, paired with `//`. `197 % 60` is `17`.

> The result takes the sign of the divisor: `7 % -2` is `-1`.

### `^`

```luau
a ^ b -> number
```

Exponentiation. `2 ^ 10` is `1024`.

> Binds tighter than unary minus, so `-2 ^ 2` is `-4` and `(-2) ^ 2` is `4`.

### `+= -= *= /= ..=`

```luau
x += 1
s ..= "!"
```

Compound assignment: read, apply, store. Also `//=`, `%=` and `^=`.

> Luau's, not Lua's. A Lua 5.1 tutorial writes `x = x + 1`, which still works everywhere.

### `== and ~=`

```luau
a == b -> boolean
a ~= b -> boolean
```

Equality and inequality. Numbers and strings compare by value.

> `~=` is Lua's spelling of "not equal"; `!=` is a syntax error. Tables compare by identity, so two tables with the same contents are not equal.

### `and, or, not`

```luau
a and b
a or b
not a
```

`and` returns its first falsy operand, otherwise the last one. `or` returns its first truthy operand, otherwise the last. `not` always returns a boolean.

> They return values, not booleans, which is what makes `local name = given or "stranger"` work — and what makes it silently replace a legitimate `false`.

### `#`

```luau
#s -> number
#t -> number
```

Length: bytes for a string, a border for a table.

> `#"héllo"` is 6 because the accented letter is two bytes; `utf8.len` answers 5. On a list with a `nil` in the middle the result is not specified.

## Strings

Strings are immutable, so every function here returns a new one. All ten also work in method form: `s:upper()`.

### `string.format`

```luau
string.format(fmt: string, ...) -> string
```

C-style formatting: `%s` any value, `%d` integer, `%.2f` fixed decimals, `%%` a literal percent.

> For plain substitution, backtick interpolation is shorter and calls `tostring` for you. `format` is worth it when you need a width or a decimal count.

### `string.sub`

```luau
string.sub(s: string, i: number, j?: number) -> string
```

The substring from `i` to `j` inclusive, 1-based. `string.sub("adventure", 1, 3)` is `"adv"`.

> Negative indices count from the end: `string.sub("adventure", -4)` is `"ture"`.

### `string.upper`

```luau
string.upper(s: string) -> string
```

An uppercased copy.

> ASCII only. Non-ASCII letters are left alone, so it is not a Unicode case fold.

### `string.lower`

```luau
string.lower(s: string) -> string
```

A lowercased copy. The usual way to compare text case-insensitively is to lower both sides.

### `string.rep`

```luau
string.rep(s: string, n: number) -> string
```

`s` repeated `n` times. `string.rep("ab", 3)` is `"ababab"`.

> Luau's version takes no separator argument. The three-argument `string.rep(s, n, sep)` from Lua 5.2 parses here but the separator is ignored.

### `string.split`

```luau
string.split(s: string, sep?: string) -> { string }
```

Splits on a plain separator (default `","`) and returns a list of the pieces.

> A Luau addition, and the reason most Luau code never needs `gmatch` to break up a line. The separator is a literal string, not a pattern.

### `string.find`

```luau
string.find(s, pattern, init?, plain?) -> (number?, number?, ...)
```

Where a pattern matches: the start and end indices, then any captures. Returns `nil` when there is no match.

> Pass `true` as the fourth argument to search for a literal string instead of a pattern — the fix when the needle contains a `.` or a `-`.

### `string.match`

```luau
string.match(s, pattern, init?) -> ...
```

What matched: the captures if the pattern has any, otherwise the whole match. `nil` when it does not match.

> The usual way to pull one field out of a line. `tonumber(string.match(s, "%d+"))` is the digits-to-number idiom.

### `string.gsub`

```luau
string.gsub(s, pattern, replacement, n?) -> (string, number)
```

Replaces every match (or the first `n`) and returns the new string **and the number of replacements**.

> Two return values, which is why `print(s:gsub(" ", "-"))` prints a count after the string. Wrap the call in parentheses to keep only the string.

### `string.gmatch`

```luau
string.gmatch(s, pattern) -> iterator
```

An iterator over every match, for use in a generic `for`: `for word in string.gmatch(line, "%a+") do`.

> It has no index: the loop variable is the match itself, not a position.

## Numbers

One number type — a 64-bit float — so `1 / 2` is `0.5` and there is no separate integer to convert to.

### `math.floor`

```luau
math.floor(x: number) -> number
```

The largest whole number no greater than `x`. `math.floor(2.7)` is `2`.

> Rounds down, not towards zero: `math.floor(-1.5)` is `-2`.

### `math.ceil`

```luau
math.ceil(x: number) -> number
```

The smallest whole number no less than `x`. `math.ceil(2.1)` is `3`.

### `math.round`

```luau
math.round(x: number) -> number
```

Rounds to the nearest whole number, halves away from zero: `math.round(2.5)` is `3` and `math.round(-2.5)` is `-3`.

> A Luau addition. Lua 5.1 code fakes it with `math.floor(x + 0.5)`, which is wrong for negatives.

### `math.abs`

```luau
math.abs(x: number) -> number
```

The distance from zero, sign discarded.

### `math.min`

```luau
math.min(...) -> number
```

The smallest of its arguments.

### `math.max`

```luau
math.max(...) -> number
```

The largest of its arguments.

> `math.max(current - damage, 0)` is the one-liner for a health value that must not go negative.

### `math.clamp`

```luau
math.clamp(x: number, min: number, max: number) -> number
```

Pins `x` into the range: `math.clamp(15, 0, 10)` is `10`.

> A Luau addition, and clearer than the `math.min(math.max(...))` sandwich it replaces.

### `math.random`

```luau
math.random() -> number
math.random(n) -> number
math.random(m, n) -> number
```

A random float in `[0, 1)`, a whole number in `[1, n]`, or a whole number in `[m, n]`.

> `math.noise` is core Luau too, not a Roblox extra — a common misconception, and the one `math` function most people assume the engine supplies.

## Tables

One container does the work of lists and records. These nine functions are all of the table library worth memorising.

### `table.insert`

```luau
table.insert(t, value)
table.insert(t, pos, value)
```

Appends to the end, or inserts at `pos` and shifts everything after it up.

> The two-argument form is the common one. The three-argument form is O(n) because of the shift.

### `table.remove`

```luau
table.remove(t, pos?) -> any
```

Removes and returns the last element, or the one at `pos`, shifting the rest down.

> Removing while counting up through the same list skips elements. Walk backwards, or build a new list.

### `table.concat`

```luau
table.concat(t, sep?, i?, j?) -> string
```

Joins a list of strings and numbers into one string. `table.concat(names, ", ")` is the way to print a list.

> Errors on any other value in the range, `nil` included — it is a list operation, not a table one.

### `table.sort`

```luau
table.sort(t, comparator?)
```

Sorts a list **in place**. The comparator returns `true` when its first argument comes first.

> The default order is byte order, so every uppercase letter sorts before every lowercase one. Case-insensitive sorting needs `function(a, b) return string.lower(a) < string.lower(b) end`.

### `table.find`

```luau
table.find(t, value, init?) -> number?
```

The index of the first `value` in the list part, or `nil`.

> A Luau addition, and the reason new Luau rarely writes a search loop by hand.

### `table.move`

```luau
table.move(a1, f, e, t, a2?) -> table
```

Copies the range `f..e` of `a1` to position `t` of `a2` (or of `a1` itself) and returns the destination.

> `table.move(src, 1, #src, #dst + 1, dst)` is how you append one list to another without a loop.

### `table.clone`

```luau
table.clone(t) -> table
```

A new table with the same keys and values.

> One level deep. A nested table is shared with the original, so a write two levels down is visible through both.

### `table.freeze`

```luau
table.freeze(t) -> table
```

Makes `t` read-only and returns it. A later write errors with `attempt to modify a readonly table`.

> Also one level deep, and permanent: there is no `table.thaw`. Freeze the constant data at the top of a module.

### `table.isfrozen`

```luau
table.isfrozen(t) -> boolean
```

Whether `t` has been frozen.

## Control and functions

The loop that walks a table you did not count, and the two ways to handle a variable number of arguments.

### `generic for`

```luau
for k, v in t do … end
for i, v in ipairs(t) do … end
for k, v in pairs(t) do … end
```

Generalized iteration: `for k, v in t` walks the array part in order and then the rest in an unspecified order. `ipairs` stops at the first `nil`; `pairs` visits every key.

> Generalized iteration is a Luau addition. Older material writes `ipairs`/`pairs` everywhere and is not wrong, just longer.

### `select`

```luau
select("#", ...) -> number
select(n, ...) -> ...
```

Counts the arguments in a `...`, or returns them from the `n`th onwards.

> `select("#", 1, nil, nil)` is `3`, where `#{1, nil, nil}` is not specified. That is the whole reason `select` exists.

### `table.pack and table.unpack`

```luau
table.pack(...) -> { n: number, ... }
table.unpack(t, i?, j?) -> ...
```

`pack` collects a `...` into a table with an `n` field holding the true count; `unpack` spreads a list back into a value list.

> The global `unpack(t)` still works. `table.unpack(t)` is the maintained spelling.

## Errors and modules

Three ways to fail on purpose, one way to survive it, and the function that loads another file.

### `pcall`

```luau
pcall(f, ...) -> (boolean, ...)
```

Calls `f` and catches any error. Returns `true` plus everything `f` returned, or `false` plus the error message. `xpcall(f, handler, ...)` is the same with a handler that runs before the stack unwinds.

> `ok` comes first, always. `local ok, value = pcall(parse, text)` reads the two cases in one line.

### `assert`

```luau
assert(v, message?) -> ...
error(message, level?)
```

`assert` returns its arguments unchanged when `v` is truthy and raises `message` when it is not. `error` raises unconditionally; `require(path)` loads another file and returns whatever it returned, once, cached for every later caller.

> Both prefix the message with the file and line unless you pass `error(msg, 0)`. `require` is not part of the language — Roblox, Lune and this sandbox each supply their own, which is why the path syntax differs between them.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Luau CLI](https://github.com/luau-lang/luau) | Run and analyse Luau outside Roblox | 0.737 | `Download from the releases page, or `brew install luau`` |
| [Lune](https://lune-org.github.io/docs) | A standalone Luau runtime with task, fs, net and process libraries | 0.10.5 | `rokit add lune-org/lune` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Editor language server: completions, type errors, go-to-definition | 1.69.0 | `Install the extension from the VS Code marketplace` |
| [StyLua](https://github.com/JohnnyMorganz/StyLua) | Formatter | 2.5.2 | `rokit add JohnnyMorganz/StyLua` |
| [Selene](https://github.com/Kampfkarren/selene) | Community linter, separate from the one inside luau-analyze | 0.31.0 | `rokit add Kampfkarren/selene` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- Lua 5.1 tutorials collect variadic arguments in a table called `arg`. Luau has no `arg`: write `...`, count it with `select("#", ...)` and collect it with `table.pack(...)`.
- `unpack(t)` as a global still works, but the maintained spelling is `table.unpack(t)`.
- `loadstring` and `dofile` appear all over Lua material. Luau removed them for sandboxing, and neither exists in the Kalo sandbox.
- `goto` and labels are common in Lua 5.2-and-later material. Luau has never added them; `continue` covers the case they are usually used for.
- Older Luau material writes `for i, v in ipairs(t)` and `for k, v in pairs(t)`. Generalized iteration means `for i, v in t` does the same job, and both older forms still work.
- `table.getn(t)` and `table.foreach(t, f)` still run but are deprecated; use `#t` and a generic `for`.
- Tutorials that say "Luau has no `os` or `debug`" are wrong: core Luau has `os.clock`, `os.date`, `os.difftime`, `os.time`, `debug.info` and `debug.traceback`. What is true is that Luau removed `io`, and that this browser sandbox exposes none of the three — so no exercise here calls them.
