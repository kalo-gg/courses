# Values and variables

*The kinds of value Luau has, how local holds one, and why nil is not the same as zero.*

Luau Fundamentals · Luau Foundations · 10 min

---

A **value** is a thing: the number `42`, the text `"adventure"`, the truth `true`. A **variable** is a name you attach to one so you can say it again later. Luau has a small, fixed set of value kinds, and `typeof` tells you which one you are holding.

**The kinds you meet first**

```luau
local n = 42
local s = "adventure"
local b = true
local nothing = nil
local t = { 1, 2, 3 }
local f = print
local v = vector.create(1, 2, 3)
local buf = buffer.create(4)

print(typeof(n), typeof(s), typeof(b), typeof(nothing))
print(typeof(t), typeof(f), typeof(v), typeof(buf))
```

Prints `number string boolean nil`, then `table function vector buffer` — one tab between each. Two more kinds exist and neither appears in this course: `thread`, which coroutines produce, and `userdata`, which only a host program such as Roblox creates.

`local` goes before a name **once**, when you first create it. After that you assign to the name without it.

A name written without `local` is a **global** instead. Globals outlive the block they were written in, are a table lookup rather than a register, and are shared between every file in the program — so a typo in one script can quietly overwrite a name in another. Writing `local` every time is the cheapest good habit in the language.

## nil is a value

`nil` is Luau's word for "no value here". It is a value in its own right, not an error and not zero: `0` is a perfectly good number and `""` is a perfectly good string.

Reading a table key that was never set gives you `nil` rather than complaining. That quietness is the source of most beginner crashes, because the program keeps running with a `nil` in hand and only falls over on the *next* line that tries to use it.

**nil, and the error it causes later**

```luau
local player = { name = "Ada" }

local title = player.title -- never set, so this is nil — not an error
print("title is", title)

print("Welcome, " .. title .. " " .. player.name)
```

Line 3 is quiet and line 4 prints `title is` and `nil`. Line 6 is where it stops: `runtime error: line 6: attempt to concatenate nil with string`. The mistake was on line 3; the error is on line 6.

There is also a plain `type` function, inherited from Lua. For the values in this course the two answer the same thing. Prefer `typeof`: inside Roblox it names engine types such as `Vector3` and `Instance`, where `type` calls all of them `userdata`.

**Key points**

- A value is a thing; a variable is a name for one, and `typeof` says which kind you hold.
- `local` goes before a name once. A name without it is a global, shared by every file.
- Assignment copies the value: `local b = a` does not link `b` to `a`.
- `nil` means "absent". Reading a key that was never set gives `nil` quietly, and the crash arrives later.
