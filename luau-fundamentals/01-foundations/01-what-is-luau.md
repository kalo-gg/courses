# What is Luau?

*A gradually typed dialect of Lua built for speed and safety, and the language Roblox runs.*

Luau Fundamentals · Luau Foundations · 9 min

---

You have probably written a Roblox script already. You may not have been told which language it was in. Tutorials say "Lua" and "Luau" as if they were the same word, and nobody stops to explain the difference.

**Luau is its own language.** It started as Lua 5.1, kept its syntax, and added a type checker, a few operators and a much faster virtual machine. Roblox maintains it and runs it in every game, but it is open source and runs perfectly well outside Roblox — including here, in this page.

## Lua, and what Luau added

| Lua 5.1 | Luau | Why it matters |
| --- | --- | --- |
| No type annotations | Gradual types, checked with `--!strict` | Mistakes are caught before the script runs |
| `..` and nothing else | String interpolation with backticks | A sentence reads like the sentence it builds, values in braces |
| No `continue` | `continue` | Leave one pass of a loop early without a flag variable |
| `x = x + 1` | `x += 1`, and `//` for floor division | Less repetition, and one less place to mistype a name |
| Interpreted | An optimising compiler and optional native code generation | Speed, which is why Roblox wrote it |

**The same sum, in both dialects**

```luau
-- Lua 5.1: no continue, no compound assignment
local total = 0
for i = 1, 10 do
	if i % 2 ~= 0 then
		total = total + i
	end
end

-- Luau: the same sum, said once
local sum = 0
for i = 1, 10 do
	if i % 2 == 0 then continue end
	sum += i
end

print(total, sum)
```

Prints `25` and `25`, tab-separated. Both add the odd numbers from 1 to 10; the second one says so without a nested block. Every Lua 5.1 program is still valid Luau, so nothing you already know stops working.

The types are **gradual**, which means optional. A file with no annotations is a perfectly good Luau file, and this course barely writes any. Add `--!strict` at the top of a file and the type checker starts objecting; leave it off and Luau infers what it can and stays quiet. Types are also erased before the program runs, so they never cost you anything at runtime — and never rescue you at runtime either.

## Where Luau runs

Luau runs in three places you are likely to meet.

- **Inside Roblox.** The engine supplies a large library on top of the language: `game`, `task`, `Instance`, `Vector3`, `workspace` and several thousand more names.
- **Standalone**, through the `luau` command-line tool or a runtime like Lune. There is no `game` here; a script is just a script.
- **Here**, in this page's sandbox. Same language, no engine.

The language is identical in all three. The *library* around it is not, and telling the two apart is the single most useful thing you can learn in this lesson.

**What this sandbox has, and what it does not**

```luau
-- The language's own libraries, everywhere Luau runs:
print(typeof(string), typeof(table), typeof(math))
print(typeof(coroutine), typeof(utf8), typeof(bit32), typeof(buffer))

-- Roblox supplies these, and this sandbox is not Roblox:
print(game, task, Instance)
```

Prints three `table`s, then four `table`s, then `nil nil nil`. An undefined global is not an error in Luau — it is `nil`. That is why a Roblox snippet pasted in here fails one line *after* the line that mentions `task`.

> **What this course does not assume**
>
> No Roblox Studio, no place file, nothing installed. Every exercise runs in this page. The sandbox is plain Luau with no library loaded, so `game`, `task` and `Instance` really are undefined here — and a lesson that would need them says so instead of pretending. Two things core Luau does have but this sandbox withholds are `os` and `debug`; no lesson uses them either.

**Key points**

- Luau is Lua 5.1 plus gradual types, `+=`, `//`, `continue`, string interpolation and a much faster VM.
- Every Lua 5.1 program is valid Luau; the reverse is not true.
- Roblox supplies the objects (`game`, `task`, `Instance`); the language supplies `string`, `table`, `math` and friends.
- This sandbox is the language alone, so an undefined global reads as `nil` rather than erroring.
