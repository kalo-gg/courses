# Safe access

*Read through values that may be nil with `?.`, fall back with `??`, insist with `!`, and find children with `->` and `=>`.*

Alloy · Foundations · 11 min

---

A shop script reads `profile.stats.coins`. Until the profile loads, `profile` is nil, and the line throws. Luau needs a guard for every step. Alloy writes the guard for you, on the same line.

## One ? guards the rest of the chain

**shop.aly**

```alloy
const profile = profiles[player.UserId] -- nil until it loads
const coins = profile?.stats.coins ?? 0
const badge = profile?.badges?[1] ?? 'Newcomer'
profile?:save()
const reward = on_reward?(coins)
```

`?.` reads a field, `?[ ]` a key, `?:` calls a method and `?( )` calls a function, each only when the value on its left is not nil. On line 2, one `?` covers `.stats.coins`: neither is read when `profile` is nil.

**What the compiler emits**

```luau
const profile = profiles[player.UserId] -- nil until it loads
local _1 = (if profile == nil then nil else profile.stats.coins) const coins = (if _1 == nil then 0 else _1)
_1 = (if profile == nil then nil else profile.badges) local _2 = (if _1 == nil then nil else _1[1]) const badge = (if _2 == nil then 'Newcomer' else _2)
if profile ~= nil then profile:save() end
const reward = (if on_reward == nil then nil else on_reward(coins)) return nil
```

Each guard is an `if … then … else` on its own source line, with temps such as `_1` for the steps between. `profile?:save()` is a statement, so it becomes a plain `if`.

`??` takes the right side only when the left is **nil**. `or` also drops `false`, so if a player turned the music off (`muted = false`), `settings.muted or true` turns it back on, while `settings?.muted ?? true` keeps the `false`.

`x ??= v` assigns only when `x` is nil, evaluating an indexed target such as `cache[key()]` once. `a!` is the reverse: it raises when `a` is nil, with the source text in the message, as in `model.PrimaryPart is nil`. Inside a chain, `!` also ends the guard.

| Write | Means |
| --- | --- |
| `a?.b`, `a?[k]` | `a.b`, `a[k]`, or nil when `a` is nil |
| `a?:m(x)`, `f?(x)` | The call, or nil when `a` or `f` is nil; then `x` is never evaluated |
| `a ?? b` | `a`, or `b` when `a` is nil |
| `a ??= b` | Assigns `b` when `a` is nil |
| `a!` | `a`, or an error naming `a` when it is nil |

## Finding children

**spawn.aly**

```alloy
const root = character->HumanoidRootPart
const gui = player=>PlayerGui
const pad = workspace->'Spawn Points'->[team_name]
```

Line 1 emits `(if character == nil then nil else character:FindFirstChild("HumanoidRootPart"))`, and line 2 emits `player:WaitForChild("PlayerGui")`. A name with spaces goes in quotes, and a name computed at run time goes in `[ ]`.

`?.` only tests for nil, and indexing a missing child still throws in Roblox. For a child that may be missing, write `a->b`. It is nil when the child is missing and nil when `a` is, so it needs no `?`. When the compiler knows `a`'s type, it checks that `a` is an Instance.

`a=>b` waits for the child. `alloy init` writes `[emit] wait_timeout = 5`, which passes a timeout to every `WaitForChild` it emits. So in a new project `=>` can return nil after five seconds, and then it guards the rest of the chain like `->`.

> **In this sandbox, => waits forever**
>
> The browser compiler has no `alloy.toml`, so `wait_timeout` is unset, as it is in any project that leaves the key out. Without a timeout, `=>` never returns nil and its result is a plain `Instance`. When a child may be missing, use `->`.

> **Recognising older code**
>
> `a and a.b`, `a:FindFirstChild('b')` and `if x == nil then x = v end` all still compile. The lints `manual_safe_access`, `manual_child_lookup` and `manual_coalesce` report each one and name the operator that replaces it.

**Key points**

- `?.`, `?[ ]`, `?:` and `?( )` give nil when the value on their left is nil, and one `?` guards the rest of the chain.
- `??` tests for nil only, so a stored `false` survives; `or` drops it. `??=` assigns only when the target is nil.
- `a!` raises with the source text when `a` is nil, and ends the guard inside a chain.
- `a->b` is a nil-safe `FindFirstChild`. `a=>b` is `WaitForChild`, nil-able when `wait_timeout` is set, as `alloy init` sets it.
