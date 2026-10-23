# Async and Futures

*`async` functions return Futures, `await` waits for them, and the combinators wait for several at once.*

Alloy · Modules, Results and async · 11 min

---

Loading a profile takes a second. In Luau you yield inside the call and hope nobody calls it from a callback, or you hand it a callback and nest the next step inside.

Alloy names the waiting. An `async function` returns a `Future<T>`, a value that settles once with the result. `await` yields until that happens.

## async and await

**profiles.aly**

```alloy
type Profile = { id: number, coins: number }

async function load_profile(id: number) -> Profile
  task.wait(1)
  return { id = id, coins = 50 }
end

async do
  const profile = await load_profile(1)
  print(profile.coins)
end
```

The declared `-> Profile` is the value inside the Future. Callers get a `Future<Profile>`, and `await` hands them the `Profile`.

**profiles.luau (ship)**

```luau
local __alloy = require("@alloy") type Profile = { id: number, coins: number }

local function load_profile(id: number): __alloy.Future<Profile> return __alloy.future(function(): Profile
  task.wait(1)
  return { id = id, coins = 50 }
end) end

__alloy.future(function()
  const profile = __alloy.await(load_profile(1))
  return print(profile.coins)
end) return nil
```

The body is wrapped in `__alloy.future(function() … end)` on the same lines. It runs on `task.spawn` under `xpcall`, and the Future keeps the result, so every later `await` reads the same value.

`await` yields the thread it runs on, so it needs a thread the runtime owns. That means an `async function` body, an `async do` block, an `after` block, the top level of a `.server.aly` or `.client.aly` script, or a function passed straight to `task.spawn` or a signal's `Connect`. Anywhere else it is `AsyncError`. A plain `function` resets the context, even inside an async one.

## Many Futures at once

Calling an async function starts it at once, so starting two loads and then waiting for both takes one second, not two. `Future.all` waits for a list. `try await` turns a rejected Future into an `Err` returned from the enclosing function.

**Combinators and try await**

```alloy
async function load_or_err(id: number) -> Result<Profile, string>
  const profile = try await load_profile(id)
  return Ok(profile)
end

const both = async do
  const pair = await Future.all([ load_profile(1), load_profile(2) ])
  const first = await Future.race([ load_profile(3), Future.delay(5) ])
  return #pair
end
```

`try await` follows the same placement rule as `await`, so its function is `async`. `async do` as an expression is a Future of the block's `return` or last line.

| Combinator | Settles with |
| --- | --- |
| `Future.all(fs)` | Every value, in order. The first failure fails it. |
| `Future.race(fs)` | The first Future to settle, success or failure. |
| `Future.any(fs)` | The first success. |
| `Future.all_settled(fs)` | One `Result` per Future, after all of them settle. |
| `Future.delay(s)` | Nothing, after `s` seconds: an awaitable `task.wait`. |

> **A rejection nobody awaits is silent**
>
> An error in an async body settles its Future as rejected, and the Future keeps the error. Nothing prints it. An `async do` statement or a call whose Future you drop loses its error that way, so `await` it, or pass an error callback to `and_then`.

**Key points**

- `async function f() -> T` returns a `Future<T>`. The body runs on `task.spawn` under `xpcall`, on the same lines.
- `await` needs an async context: an async body, `async do`, `after`, a script's top level, or a function passed to `task.spawn` or `Connect`. A plain `function` resets it.
- Start every Future first, then `await Future.all(…)`. `race`, `any`, `all_settled` and `delay` cover the other waits.
- `try await` turns a rejection into an `Err` from the enclosing function.
- A rejected Future that nobody awaits keeps its error silently.
