# Writing readable Luau

*Naming, locals over globals, small functions, and comments that say why rather than what.*

Luau Fundamentals · Modules & Mistakes · 9 min

---

Every piece of code you write is read more often than it is written, usually by you, usually six weeks later, usually while something is broken. The habits below are not style opinions — each one removes a class of bug or a class of confusion.

| Habit | Instead of | Because |
| --- | --- | --- |
| Name for the thing, not the type | `local t`, `local x`, `local data` | `local rooms` tells a reader what is in it; `local t` makes them scroll |
| `local` everything | A bare `name = …` | A global is a table lookup, is shared by every file, and turns a typo into a silent new variable |
| One job per function | A 60-line function with three sections | Sections that need a comment saying what they do want to be functions with that name |
| Guard at the top, work at the bottom | Deeply nested `if`s | The reader learns what the function refuses before they read what it does |

**The same function, twice**

```luau
-- Dense
local function f(t, k)
	if t and t[k] and t[k].e then return t[k].e.n end
	return nil
end

-- Readable
local function northExitOf(rooms, roomKey)
	local room = rooms and rooms[roomKey]
	if room == nil or room.exits == nil then
		return nil
	end
	return room.exits.north
end
```

Same behaviour, same number of decisions. The second one names the two things involved, so a reader who has never seen the rooms table can still tell what a missing exit means here.

The `local` habit deserves its own paragraph. A local is stored in a register, which is fast; a global is a hash lookup on every read, which is not. More importantly, a global is visible everywhere. Two files that both forget a `local` on a variable called `count` will quietly share it, and the bug that follows looks like action at a distance.

**A global outlives its block; a local does not**

```luau
do
	count = 1       -- no local: this is a global
	local shown = 2 -- local to this block
end

print(count) -- 1: the global is still here
print(shown) -- nil: the local ended with its block
```

That asymmetry is the whole argument for `local`. The global survived a block it was never meant to leave, and nothing in the source says so — you have to notice the missing keyword.

## Comments that earn their line

Write **why**, not **what**. The code already says what it does; a comment that repeats it is one more thing to keep in sync, and it will not be kept in sync.

`-- add 1 to count` above `count += 1` is noise. `-- levels below 3 have no market, so the greeting skips it` is information the code cannot carry. The test is simple: if you deleted the comment, would anything be lost?

> **Tooling does the rest**
>
> None of this needs to be argued about in review. **StyLua** formats Luau to one style so indentation and line breaks stop being opinions. **Selene** lints for shadowed variables, unused locals and accidental globals. And `luau-analyze`, which ships with the Luau CLI, has its own linter and type checker built in. Kalo's Larvae course sets all three up in one config.

**Key points**

- Name for the thing, not the type, and prefer the shortest name that still identifies it.
- `local` everything: globals are slower, shared between files, and turn typos into new variables.
- One job per function; a section that needs a heading comment wants to be its own function.
- Comments say **why**. If deleting the comment loses nothing, delete it.
- StyLua, Selene and `luau-analyze` enforce the mechanical half so review can be about the rest.
