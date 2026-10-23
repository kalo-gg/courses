# Match

*Take a value apart with `match`, as a statement or an expression, and let the compiler reject a match that misses a variant.*

Alloy · Structs and enums · 11 min

---

An if-chain over `msg.tag` works until someone adds a variant. The new case falls through every branch without an error, and the bug shows up far from the enum that changed.

`match` tests the variant and binds its payload in one arm. A `match` with no `default` must name every variant, and the compiler checks that it does.

## Taking an enum apart

**area.aly**

```alloy
enum Shape
  Circle(number)
  Rect(number, number)
end

local function area(shape: Shape) -> number
  return match shape with
    case Circle(r) then math.pi * r * r
    case Rect(w, h) then w * h
  end
end
```

As an expression, each arm's value follows `then`. `r`, `w` and `h` bind the payloads. There is no `default`, because the two arms cover both variants.

**area.luau (ship)**

```luau
local __alloy = require("@alloy") local Shape = {} Shape.__index = Shape
function Shape.Circle(_1) return setmetatable({ tag = "Circle", _1 = _1 }, Shape) end
  function Shape.Rect(_1, _2) return setmetatable({ tag = "Rect", _1 = _1, _2 = _2 }, Shape) end
function Shape.is(v) return (type(v) == "table" and getmetatable(v) == Shape) end Shape.__tostring = function(v) return __alloy.show_variant("Shape", v) end type Shape = typeof(setmetatable({} :: { tag: "Circle", _1: number }, Shape)) | typeof(setmetatable({} :: { tag: "Rect", _1: number, _2: number }, Shape))

local function area(shape: Shape): number
  return (
    if type(shape) == "table" and shape.tag == "Circle" then math.pi * shape._1 * shape._1
    elseif type(shape) == "table" and shape.tag == "Rect" then shape._1 * shape._2 else error("match: no arm covers this value", 2)
  )
end return nil
```

The arms become an if-chain on `tag`, inside parentheses, on the lines the arms used. The final `else error(...)` fires only for a value that is not a `Shape` at all.

Delete the `Rect` arm and the file stops compiling with `error(4.2): ExhaustiveMatch`. The message names the gap: *`Shape` has no arm for `Rect`*. So when you add a variant, every `match` without a `default` turns up as work to do.

That is also why the `unreachable_default` lint asks you to delete a `default` when the arms already cover every variant. When you do need a `default`, it comes last.

## Patterns

A `case` can match more than a variant. Literals, struct shapes and arrays match too. They have no list of variants to cover, so a `match` on a number, a struct or an array always needs a `default` or a catch-all binding.

| Pattern | Matches |
| --- | --- |
| `Rect(w, h)` | That variant, binding its payloads; `_` skips one |
| `100`, `'a'`, `nil` | A value equal to the literal |
| `Vec2 { x = 0, y }` | A `Vec2` whose `x` is 0, binding `y` |
| `[first, ...rest]` | An array with at least one item; `rest` is the tail |
| `Kick(_, _) or Quit` | Either side; both sides must bind the same names |
| `s where s >= 50` | Anything, bound to `s`, when the guard holds |

**handle.aly**

```alloy
enum Msg
  Chat(string)
  Kick(string, number)
  Quit
end

local function handle(msg: Msg) -> ()
  match msg as m with
    case Chat(text) where #text > 200 then
      print('too long')
    case Chat(text) then
      print(`chat: {text}`)
    case Kick(_, _) or Quit then
      print('leaving:', m)
  end
end

local function grade(score: number) -> string
  return match score with
    case 100 then 'perfect'
    case s where s >= 50 then
      print('passed with', s)
      'pass'
    default 'fail'
  end
end
```

`as m` names the value for every arm. An arm can run statements before its value. In `grade`, the arm's last line, `'pass'`, is its value, and the emit moves the `return` into each arm. A number has no variants, so `grade` needs its `default`.

> **A bare name binds; it does not compare**
>
> `case target then`, with `target` a local in scope, is a new binding that matches every value, so the arms after it never run. The `pattern_shadows_local` lint warns about it. To compare, write `case v where v == target`.

> **Recognising older code**
>
> `case s and s >= 50` is the older spelling of a guard. It still parses, and the `match_guard_and` lint rewrites it to `case s where s >= 50`.

> **A one-off test**
>
> `$matches(shape, Circle(_))` is `true` when the value fits the pattern. Use it when you need a yes-or-no answer rather than a whole `match`.

**Key points**

- `match` works as a statement or an expression, and an arm's last line is its value.
- With no `default`, every variant needs an arm, or the compile fails with `error(4.2): ExhaustiveMatch`.
- A guarded arm proves nothing about coverage, and `default` always comes last.
- A match on a literal, struct or array always needs a `default` or a catch-all binding.
- Delete a `default` the arms already cover (`unreachable_default`), so the next new variant is a compile error.
