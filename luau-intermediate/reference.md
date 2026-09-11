# Luau Intermediate reference

*Luau Intermediate 0.733, released August 7, 2026. Checked on September 9, 2026.*

---

## Annotations

The colon, the cast, and the three types that stand for "anything", "something" and "nothing".

### `local x: T`

```luau
local qty: number = 2
```

Constrains what may ever be assigned to a local. Rarely needed for an obvious literal; nearly always needed for an empty table.

### `parameter: T`

```luau
function f(item: string, qty: number)
```

The contract callers are checked against. Under `--!strict` an unannotated parameter is an error rather than an implicit `any`.

### `): T`

```luau
function f(s: string): (boolean, string?)
```

The return type, written after the closing parenthesis. One value may be bare; two or more are parenthesised, and `()` means the function returns nothing.

### `::`

```luau
value :: string
```

A cast: asserts a type the checker could not prove and takes your word for it. Erased at run time like every other annotation.

> A cast to a specific type is a claim you can be wrong about, in one place. A cast to `any` is not a claim at all.

### `any`

```luau
local x: any
```

Opts a value out of checking entirely, in both directions. Everything computed from an `any` is itself `any`.

> The escape hatch the whole first half of this course is about avoiding; `unknown` is nearly always the honest alternative.

### `unknown`

```luau
local raw: unknown = readSetting()
```

Accepts any value going in and permits nothing coming out until it is narrowed or cast. The type for a value you genuinely do not know yet.

### `never`

```luau
function fail(msg: string): never
```

The type with no values: the return type of a function that always throws, and what a variable narrows to when every case has been ruled out.

## Modes and directives

A comment on the first line of a file, applying to that file only.

### `--!nocheck`

```luau
--!nocheck
```

No inference and no reports for this file. For generated or vendored code you will not edit.

### `--!nonstrict`

```luau
--!nonstrict
```

Infers types but reports only what will certainly misbehave; an unannotated parameter is given the type `any`.

> **This is the default.** A file with no directive is checked in nonstrict, not in strict and not in nocheck — plenty of tutorials say otherwise.

### `--!strict`

```luau
--!strict
```

Reports what might misbehave: missing fields, unnarrowed optionals, wrong returns, and unannotated parameters.

### `--!nolint`

```luau
--!nolint
--!nolint LocalShadow
```

Silences the linter, which is a separate tool from the type checker. With a rule name it silences one rule.

## Table and value types

Records, indexers, optionals and the unions that combine them.

### `record type`

```luau
type Item = { name: string, qty: number }
```

Lists exactly the fields the table has. A missing required field and an extra unlisted field are both errors, which is what catches a misspelled read.

### `{ T }`

```luau
type Names = { string }
```

An array of `T`; shorthand for `{ [number]: T }`.

### `{ [K]: V }`

```luau
type Counts = { [string]: number }
```

An indexer: any key of type `K` maps to a `V`. Says nothing about which keys exist, so prefer a record wherever the fields are known.

### `read`

```luau
type Config = { read id: string, retries: number }
```

Marks a field read-only: callers may look at it and may not assign to it.

> A checker-only guarantee. Nothing stops the write at run time, exactly as nothing enforces any other annotation.

### `T?`

```luau
local tag: string?
```

Shorthand for `T | nil`. The checker refuses to use the value until the code has ruled `nil` out, by an `if` test or an `or` default.

### `A | B`

```luau
type Value = number | string
```

A union: the value is one of several types, and only operations valid for every member are allowed until it is narrowed.

### `A & B`

```luau
type Named = { name: string } & { qty: number }
```

An intersection: a value satisfying both types at once. Used for extending a record type without rewriting it.

### `string literal types`

```luau
type Cmd = "add" | "remove"
```

A string literal is a type with one value in it. Combined into a union it gives a checked set of options, and a shared literal-typed field is what makes a tagged union work.

> Booleans have singleton types too (`true`, `false`). **Numbers do not**: `type N = 1` is a syntax error in 0.733, not a checker complaint.

### `typeof`

```luau
type Acc = typeof(setmetatable({} :: Data, Account))
```

The type of an expression, computed without evaluating it. The documented way to type the class pattern.

## Aliases and generics

Naming a shape, sharing it, and leaving a hole in it.

### `type`

```luau
type Item = { name: string }
```

Names a shape; the checker substitutes it wherever the name appears. Visible in the declaring file only.

### `export type`

```luau
export type Item = { name: string }
```

Makes the alias reachable from another file, through the name that file required the module under: `local types = require("./types")` then `types.Item`.

> There is no global type namespace. The declaring file owns the name, and a module of nothing but types still has to return a value.

### `alias parameter default`

```luau
type Result<T, E = string> = …
```

A type parameter with a fallback, so `Result<number>` is legal and means `Result<number, string>`.

> Aliases may do this and **functions may not**: `function f<T, E = string>()` does not parse.

### `<T>`

```luau
function first<T>(list: { T }): T?
```

A type parameter the caller decides. Using it in both a parameter and the return is what ties the two ends together, which is the difference from `any`.

### `explicit instantiation`

```luau
id<<number>>(42)
```

States the type argument instead of leaving it to inference, for the rare call with nothing to infer from. The doubled angle brackets avoid ambiguity with `<`.

### `...T`

```luau
function sum(...: number): number
```

A variadic whose arguments are all of one type. Fixed parameters come first and nothing may follow the pack.

### `T...`

```luau
function wrap<A..., R...>(f: (A...) -> R...): (A...) -> R...
```

A generic type pack: a list of types the caller decides, for wrapping a function whose signature you do not know.

> A pack is a list of types, not a type, so it cannot annotate a local or a field — only the variadic position.

### `arrow type`

```luau
type Combine = (number, number) -> number
```

The type of a function value. Typing a callback parameter is what lets the lambdas passed to it infer their own parameters.

## Metatables

The hooks a table consults when the ordinary rules run out.

### `setmetatable`

```luau
setmetatable(t: T, mt: table?): T
```

Attaches a metatable and returns the table, which is why it is usually written as part of the expression that creates it.

### `getmetatable`

```luau
getmetatable(t): table?
```

Returns the attached metatable, or `nil`. In the class pattern this is the class table, which is how an instance check is written.

### `rawget`

```luau
rawget(t, key)
```

Reads a key with the metatable ignored: what is actually stored, rather than what the table answers.

### `rawset`

```luau
rawset(t, key, value)
```

Writes a key with the metatable ignored.

> Not optional inside a `__newindex` hook: an ordinary assignment there re-enters the hook and never terminates.

### `rawequal`

```luau
rawequal(a, b): boolean
```

Compares two values without consulting `__eq`. Identity, not equality.

### `__index`

```luau
__index = table | function(t, key)
```

Consulted when a read misses. A table means "look the key up there instead"; a function is called with the table and the key.

> Consulted on a miss and never on a hit, which is why a metatable can supply what a table lacks and can never override what it has.

### `__newindex`

```luau
__newindex = table | function(t, key, value)
```

Consulted when a key that is **not already present** is written. Used for read-only views and change tracking.

> Fires only for absent keys, so a write to an existing key never reaches it — and after a `rawset` inside the hook, that key is present forever. To intercept every write, keep the exposed table empty.

### `__tostring`

```luau
__tostring = function(t): string
```

Used by `tostring(t)`, which `print` calls for you. The cheapest way to make a class debuggable.

### `__eq`

```luau
__eq = function(a, b): boolean
```

Used by `==` when both operands are tables that share the metamethod. Never called when one side is not a table.

### `__call`

```luau
__call = function(t, ...)
```

Lets a table be called like a function, with the table itself as the first argument.

## Iteration

The protocol a generic for speaks, and the ways to satisfy it.

### `generic for`

```luau
for a, b in iterator, state, control do
```

Calls the iterator until its first returned value is `nil`. That is the entire protocol; `false` is a perfectly good loop value.

### `generalized iteration`

```luau
for k, v in t do
```

A Luau extension: a plain table may be looped with no helper at all. The array part comes in order and everything after it does not.

### `__iter`

```luau
__iter = function(self)
```

Called once per loop, with the object, so a generic `for` can iterate it directly. Returns whatever the loop accepts, usually one closure.

> Supersedes `__pairs` and `__ipairs`, which Luau does not have and will not add. Build the iterator inside `__iter` so a second loop starts fresh.

### `pairs`

```luau
pairs(t) -> (next, t, nil)
```

Returns the three-value form for every key of a table. Still worth knowing, though generalized iteration has made it unnecessary for looping.

### `ipairs`

```luau
ipairs(t) -> (iterator, t, 0)
```

Walks `1, 2, 3 …` until the first `nil`, so it stops at a hole.

> Ignores metamethods, deliberately, as do `#` and the whole table library. A proxy whose values only exist through `__index` iterates as empty.

### `next`

```luau
next(t, key?) -> (key, value)?
```

The stateless step `pairs` returns: given a table and a key, hands back the following pair, or nothing at the end.

## Modules and errors

The boundary between files, and the two ways a function reports that something went wrong.

### `require`

```luau
local types = require("./types")
```

Loads a module and returns the one value it returned. Everything above that value in the file is private by construction.

> Resolves inside a project, where the other files are its siblings. A single-file lesson exercise has nothing to require, so those exercises are self-contained.

### `pcall`

```luau
pcall(f, ...) -> (boolean, ...)
```

Calls a function and catches an error, returning success and either the results or the message.

> A `pcall` around your own module's routine failure is a sign the module should have returned that failure instead of throwing it.

### `error`

```luau
error(message, level?)
```

Throws. For programmer mistakes and unreachable states, where the traceback is the point.

> The message is prefixed with a source position, so a thrown string is not something to show a player.

### `assert`

```luau
assert(value, message?) -> value
```

Throws when the value is falsy and returns it otherwise. Three of these at the bottom of a file is a test suite.

> Assert on the value, not on the absence of an error: `pcall(f)` followed by `assert(ok)` is a test that cannot fail.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Language server — the underlines this course is about | 1.69.0 | `VS Code marketplace: JohnnyMorganz.luau-lsp` |
| [Luau CLI](https://github.com/luau-lang/luau) | The interpreter and `luau-analyze`, the checker on the command line | 0.737 | `rokit add luau-lang/luau` |
| [Lune](https://github.com/lune-org/lune) | Standalone Luau runtime with a filesystem, network and process API | 0.10.5 | `rokit add lune-org/lune` |
| [StyLua](https://github.com/JohnnyMorganz/StyLua) | Formatter | 2.5.2 | `rokit add JohnnyMorganz/StyLua` |
| [Selene](https://github.com/Kampfkarren/selene) | Linter | 0.31.0 | `rokit add Kampfkarren/selene` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- Tutorials that call `--!strict` "the default". It is not and never has been: a file with no directive is checked in **nonstrict**, which silently gives unannotated parameters the type `any`.
- Material describing the old type solver's inference as current. Luau's new solver changed what is inferred for empty tables, generic calls and narrowing; if an article's examples predate it, treat its inference claims as history.
- `__pairs` and `__ipairs`, from Lua 5.2 material. Luau has neither and has said it will not add them — `__iter` is the one metamethod for iteration, and it drives the generic `for` alone.
- Examples using a `class` keyword from experimental Luau builds. The syntax is flagged off in 0.733; the class pattern is still a table, `__index` pointing at itself, and a `new` that calls `setmetatable`.
- `<const>` attributes copied from Lua 5.4. Luau spells a compile-time constant `const x = 1` and applies it in different places; the angle-bracket attribute syntax does not parse.
- OOP tutorials whose methods take an unannotated `self`. Strict mode requires the annotation, because Luau considered a shared self type and decided against it — every method spells the receiver's type out.
