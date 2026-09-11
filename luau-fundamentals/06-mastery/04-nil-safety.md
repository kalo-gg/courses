# Guarding against nil

*Most crashes are a nil used one line after it appeared, and four small habits prevent nearly all.*

Luau Fundamentals · Modules & Mistakes · 10 min

---

Count up the errors in this course so far and nearly all of them are the same error: something was `nil`, nothing complained, and the crash arrived on a later line that was perfectly correct.

That is not a flaw to be endured. It is a pattern, and a pattern has fixes. This lesson is the four of them.

**The crash, and where it actually started**

```luau
local rooms = {
	hall = { name = "the hall", exits = { north = "library" } },
	cellar = { name = "the cellar" },
}

local key = "attic"        -- 1. the value that is wrong
local room = rooms[key]    -- 2. nil, quietly
print("moving to " .. room.name) -- 3. the line that errors
```

Three lines, three different roles. Line 5 is the mistake, line 6 is where it became a `nil`, and line 7 is what the traceback will name. Fixing line 7 fixes nothing.

## Four guards

| Guard | Looks like | Fits when |
| --- | --- | --- |
| Early return | `if room == nil then return end` | The function has nothing useful to do without it |
| `or` default | `local title = p.title or "none"` | There is a sensible stand-in — and `false` is not a legitimate value |
| `and` chain | `t and t.exits and t.exits.north` | You are reading two or more levels down and absence is normal |
| `assert` at the boundary | `assert(room, "no such room: " .. key)` | The caller broke a contract and should hear about it immediately |

The one that needs judgement is the last. `assert` belongs at the **edge** of a module — the public function a caller reaches first — and not in the middle of it. An assertion deep inside a private helper fires long after the bad value entered, which puts you right back to reading a traceback for where it came from.

And the `or` default carries the warning Module 2 already gave you: `x or default` cannot tell `nil` from `false`. When `false` is a value the field could legitimately hold, the guard has to be `if x == nil`.

**The same read, guarded**

```luau
local function exitFrom(key, direction)
	local room = rooms[key]
	if room == nil then
		return "no such room"
	end
	if room.exits == nil or room.exits[direction] == nil then
		return "no exit that way"
	end
	return room.exits[direction]
end

print(exitFrom("hall", "north"))   -- library
print(exitFrom("cellar", "north")) -- no exit that way
print(exitFrom("attic", "north"))  -- no such room
```

Two early returns and then the work, with the two failures distinguished. A single `and` chain would have collapsed both into one `nil`, which is fine when the caller does not care which happened — and this one does.

One last habit that is not on the list because it is not a guard: **convert at the edge**. A value arriving from outside — a command string, a save file, a network payload — should be turned into the kind you want, and checked, at the moment it arrives. Every guard you write further in is a guard you would not have needed.

**Key points**

- Nearly every crash is a `nil` produced on one line and used on a later one.
- Four guards: early return, `or` default, `and` chain, and `assert` at the boundary.
- `assert` belongs at a module's edge, where the bad value entered, not deep inside it.
- `or` cannot tell `nil` from `false`; test `== nil` when `false` is legitimate.
- Convert and check values at the edge and most of the guards further in disappear.
