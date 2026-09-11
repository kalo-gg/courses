# Printing, and reading an error

*print and warn are your output, and a stack traceback tells you which line actually broke.*

Luau Fundamentals · Luau Foundations · 9 min

---

`print` is the entire debugging toolkit at this stage, and it stays useful long after you have better ones. It takes any number of arguments, converts each to text and puts a **tab** between them. `warn` does the same thing but marks the line, so an important message stands out in a wall of output.

**print and warn**

```luau
print("plain")
print("Ada", 42, true)
warn("that took a while")
```

Prints `plain`, then `Ada`, `42` and `true` with a tab between each, then `[warn] that took a while` in this sandbox (in Studio the warn line is orange). The separator between arguments is a tab, not a space — worth knowing before you try to match printed output against a string.

## When it goes wrong

An error stops the script and prints a **traceback**: the message first, then the call stack, innermost call at the top. Read it top to bottom and it tells you the story backwards — where it broke, then who asked for it, then who asked for that.

The line number in the message is the line that *threw*. That is not always the line with the mistake, which is the single most important thing to know about reading one.

**A traceback, read from the top**

```luau
local player = { name = "Ada" }

local function greet(who)
	print("Hello, " .. who.title .. " " .. who.name)
end

local function start()
	greet(player)
end

start()
```

The sandbox prints:

`runtime error: line 4: attempt to concatenate nil with string`
`stack traceback:`
`line 4 function greet`
`line 8 function start`

Line 4 is where it threw. But line 4 is fine: the bug is that `player` on line 1 has no `title`, so `who.title` is `nil`.

> **Print the thing, not "here"**
>
> `print("here")` tells you the line ran, which you could already guess. `print("who.title is", who.title, typeof(who.title))` tells you what the value is and what kind it is, in one line, which is usually the whole answer. Print values, not milestones.

There is a second reason to print the type as well as the value. `print(x)` for the string `"42"` and the number `42` produce exactly the same two characters on screen. If your bug is that a number arrived as text, printing the value alone will show you nothing wrong at all.

**Key points**

- `print` takes any number of arguments and separates them with a tab; `warn` marks the line.
- An error prints a message and a traceback, innermost call first.
- The line in the message is where it threw, which is often not where the mistake is.
- Print the value and its type together: `"42"` and `42` look identical on screen.
