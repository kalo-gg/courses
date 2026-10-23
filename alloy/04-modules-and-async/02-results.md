# Results and try

*Return failure as a value, pass it up with `try`, and catch Roblox's throws once, at the edge.*

Alloy · Modules, Results and async · 11 min

---

`tonumber('abc')` returns nil. A DataStore call throws. Luau leaves you two ways to fail, and callers forget the nil check or wrap things in `pcall` and test `ok` by hand.

In Alloy, a function that can fail says so in its return type. `Result<T, E>` is either `Ok(value)` or `Err(error)`, and the caller has to open it to reach the value.

## Failure as a value

**shop.aly**

```alloy
function find_item(id: string) -> Result<Item, string>
  const item = catalog:get(id)
  return item ? Ok(item) : Err(`no item called {id}`)
end

function buy(coins: number, id: string) -> Result<number, string>
  const item = try find_item(id)
  if item.price > coins then
    return Err(`{id} costs {item.price}`)
  end
  return Ok(coins - item.price)
end
```

`try find_item(id)` unwraps an `Ok`, or returns the `Err` from `buy` at once. It works only inside a function that returns a `Result`; anywhere else it is `ResultError`. `Result`, `Ok` and `Err` need no import.

**buy, emitted (lines 6–12)**

```luau
local function buy(coins: number, id: string): __alloy.Result<number, string>
  local _1 = find_item(id) if _1.tag == "Err" then return _1 end const item = _1._1
  if item.price > coins then
    return __alloy.Err(`{id} costs {item.price}`)
  end
  return __alloy.Ok(coins - item.price)
end return nil
```

The `try` line stays one line: a temp, a tag test and an early return. A Result is a tagged table, `{ tag = "Ok", _1 = value }`.

To read a Result, `match` both cases, or bind the success with `if const Ok(v) = r`. The methods cover the short cases: `unwrap_or(d)` gives a default, `map(f)` changes the value, and `and_then(f)` chains another step that can fail. A call whose Result nobody reads draws the `dropped_result` lint, because its failure would pass in silence.

**Reading a Result**

```alloy
match buy(50, 'sword') with
  case Ok(left) then print('bought, coins left:', left)
  case Err(reason) then warn(reason)
end

if const Ok(left) = buy(30, 'sword') then
  print(left)
end

const coins = buy(50, 'gem'):unwrap_or(50)
```

## Throws at the edge

Roblox APIs throw, so catch them once, where your code meets them. `Result.pcall(f, ...)` gives an `Ok` of the return or an `Err` of the error, with the traceback attached. `try do … end` does the same for a whole block. From there, Results flow inward.

Some failures mean the code itself is wrong, and those should still throw with `error` or `assert`. Ask whether a correct caller with valid data could reach the line. If not, throw.

**The boundary**

```alloy
import { DataStoreService } from '@game'

const store = DataStoreService:GetDataStore('coins')

function load_coins(key: string) -> Result<number, string>
  const saved = try Result.pcall(store.GetAsync, store, key)
  return Ok(saved ?? 0)
end
```

One `Result.pcall` at the call that throws, then `try` to pass any failure up. `try do … end` emits `__alloy.try_block(function(__fail) … end)`, and a throw inside it becomes an `Err`.

> **Recognising older code**
>
> `local ok, err = pcall(f)` still compiles. The `raw_pcall` lint reports it, because the flag has to be tested by hand and the error loses its traceback. Write `Result.pcall` at the boundary, `try do … end` around a block, or return a `Result`.

**Key points**

- A function that can fail returns `Result<T, E>`: `Ok(value)` or `Err(error)`. None of the three needs an import.
- `try expr` unwraps an `Ok` or returns the `Err`, and only inside a function that returns a `Result`.
- Read a Result with `match`, `if const Ok(v) = r`, or `unwrap_or`, `map` and `and_then`.
- Wrap a throwing Roblox call once with `Result.pcall`, or a block with `try do … end`, then let Results flow inward.
- Throw only when the code is wrong. Expected failures are `Err` values.
