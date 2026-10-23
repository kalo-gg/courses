# Conditional bindings

*Bind a value and test it in one step with `if const`, `while const`, the guard clause and the refutable `const`, and know what each does when the test fails.*

Alloy · Structs and enums · 10 min

---

Half of Roblox code is nil checks: `local character = player.Character`, then `if character then`. The name outlives the check, so a later line can read it while it is still nil.

A **conditional binding** names a value and tests it on one line, and the name exists only where the test passed.

## Bind and test in one step

**spawn.aly**

```alloy
local function place(player: Player, spawn: CFrame) -> ()
  if const character = player.Character then
    character:PivotTo(spawn)
  end
end

local function boost(player: Player) -> ()
  if const body = player.Character; local humanoid = body->Humanoid where humanoid.Health > 0 then
    humanoid.WalkSpeed = 24
  elseif const backpack = player->Backpack then
    print('no character yet; backpack holds', #backpack:GetChildren())
  end
end
```

Write `if const` when the branch only reads the name (the `prefer_const` lint asks for it), and `if local` when it reassigns the name or writes into its value, as `boost` does to `humanoid`. `;` chains a second binding that can use the first, and `where` adds a condition to the binding before it.

**place, compiled (ship, lines 1–5)**

```luau
local function place(player: Player, spawn: CFrame): ()
  do local _c1 = player.Character if _c1 then local character = _c1
    character:PivotTo(spawn)
  end end
end
```

A temporary holds the value, and `character` is declared only inside the branch, so nothing after `end` can see it.

## When the binding fails

Sometimes failing is the short path. The **guard clause** `if not const item = find(id) then return end` runs its block when the binding fails, and keeps `item` in scope after it.

A variant pattern binds the same way. `const Move(n) = cmd else … end` takes the payload, and the `else` block must leave with `return`, `break`, `continue` or an error. Version 0.1.0 does not check that it leaves, so a block that falls through leaves `n` as nil. Without `else`, any other variant raises an error that names the pattern.

**guards.aly**

```alloy
enum Cmd
  Move(number)
  Wait
end

local function find(items: { [string]: number }, id: string) -> number
  if not const count = items[id] then
    return 0
  end
  return count
end

local function steps(cmd: Cmd) -> number
  const Move(n) = cmd else
    return 0
  end
  return n
end
```

`count` and `n` stay in scope after their guards, because the failing path has already returned.

`while const` keeps looping as long as the binding holds. Its `where` is part of that test, so the first value that fails it **ends** the loop, after that value has been taken. To skip items instead, filter a `for` loop, which can also destructure in its head.

**jobs.aly**

```alloy
const jobs = { { id = 1, ready = true }, { id = 2, ready = false }, { id = 3, ready = true } }

for _, { id, ready } in jobs where ready do
  print('ran', id)
end

while const job = table.remove(jobs, 1) where job.ready do
  print('took', job.id)
end
print(#jobs, 'left')
```

The `for` prints `ran 1` and `ran 3`. The `while` takes job 1, then takes job 2, which fails the `where`, and stops. Job 2 is gone, and job 3 is still queued: the last line prints `1 left`.

**Key points**

- `if const x = e then` binds and tests at once, and `x` exists only in the branch. Use `if local` when the branch reassigns `x` or writes into it.
- Chain bindings with `;`, and filter each one with `where`.
- `if not const x = e then return end` keeps `x` in scope after the guard.
- `const Move(n) = cmd else … end` takes a payload, and its `else` block must leave.
- A `while const … where` loop stops at the first failure, while a `for … where` loop skips it.
