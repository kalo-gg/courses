# 5. Functions & Scope

*Package behaviour, return more than one value, take any number of arguments, and capture state.*

Luau Fundamentals · Medium

---

A function is a value like any other: it can be stored in a variable, passed to another function and returned from one. This module covers declaring and calling them, the multiple return values that make Luau's standard library feel unlike most languages, variadic arguments, and the scope rules that decide which names a function can see and how long they live.

## Lessons

1. [Defining and calling](01-defining-functions.md) — Declare with local function, call with parentheses, and pass functions around like any value.
2. [Returning more than one value](02-multiple-returns.md) — Luau functions return any number of values, and where you put the call decides how many survive.
3. [Any number of arguments](03-varargs.md) — The ... expression, select for counting, and table.pack when a nil has to be preserved.
4. [Scope and closures](04-scope-and-closures.md) — Where a local is visible, how long it lives, and what a function captures when it is created.
