# Structs and impl

*Declare a named shape with defaults, construct it with `new` so the compiler checks every field, and give it methods with `impl`.*

Alloy · Structs and enums · 10 min

---

In Luau a class is a convention: a table, `Wallet.__index = Wallet`, and a `new` that sets the fields. Nothing stops `{ coin = 5 }` from reaching code that reads `coins`. Nothing notices a missing field until a player reaches the line that reads it.

A **struct** gives the shape a name. The compiler knows its fields, so it checks every construction before the game runs.

## A struct and its defaults

**vec2.aly**

```alloy
struct Vec2
  x: number
  y: number = 0
end

const v = new Vec2 { x = 3 }
print(v)
```

`y` has a default, so `new Vec2 { x = 3 }` is complete. `print(v)` shows `Vec2 { x = 3, y = 0 }`: every struct gets a `__tostring` that prints its name and fields.

**vec2.luau (ship)**

```luau
local __alloy = require("@alloy") local Vec2 = {} Vec2.__index = Vec2 setmetatable(Vec2, { __call = function(_, f) if f.y == nil then f.y = 0 end return setmetatable(f, Vec2) end }) function Vec2.new(f) return Vec2(f) end

 type Vec2 = typeof(setmetatable({} :: { x: number, y: number }, Vec2)) Vec2.__tostring = function(s) return __alloy.show_struct("Vec2", s, { "x", "y" }) end

const v = Vec2({ x = 3 })
print(v) return nil
```

Seven lines in, seven out. The class table, the `__call` that fills in defaults, the type and the `__tostring` all sit on lines the declaration already used, so an error on line 6 points at your line 6. A module that exports nothing ends with `return nil`.

Each of those mistakes is a compile error from book section 3.6, so nothing runs until it is fixed. `new Name { … }` is the only way to build a struct. The bare forms are rejected too, so the compiler can check every construction.

| You write | The compiler reports |
| --- | --- |
| `new P { }` (with `x` unset) | `ConstructorError`: `new P { ... }` leaves `x` unset |
| `new P { x = 1, z = 2 }` | `StructError`: `P` has no field `z` |
| `P { x = 1 }` | `ConstructorError`: construct `P` with `new P { ... }` |
| `P(1)` | `ConstructorError`: `P(...)` is not a constructor |

For a struct `P` with one field, `x: number`. Each one prints as `error(3.6)`, and `alloy doc 3.6` explains it.

## Methods with impl

**timer.aly**

```alloy
struct Timer
  seconds: number
  private left: number = 0
end

impl Timer
  function new(seconds: number) -> Timer
    return new Timer { seconds = seconds, left = seconds }
  end

  function tick(self, dt: number) -> boolean
    self.left = math.max(self.left - dt, 0)
    return self.left == 0
  end
end

const t = new Timer(3)
print(t:tick(1), t:tick(2))
```

The impl's `new` takes no `self`, so `new Timer(3)` calls it (the ship writes `Timer.new(3)`). `tick` takes `self` and is called with `:`. This prints `false true`.

A function in an `impl` that does not take `self` is a **static**, called with a dot. Calling one with a colon, as in `Timer:new(3)`, passes the table as an extra first argument, and the `static_call` lint warns about it.

`private` marks a member that only the struct's own `impl` should touch, and `read` marks a field that nobody should write. Neither one emits anything. The type checker enforces them, and the `private_access` lint reports outside access in the same file.

> **`class` is not here yet**
>
> Alloy 0.1.0 parses the `class Name … end` syntax from the Luau classes RFC, then reports `StructError: class is parsed and not compiled yet` and blanks the block. A `struct` with an `impl` is the form that runs.

> **Recognising older code**
>
> A hand-written class, `Wallet.__index = Wallet` with a `Wallet.new` that calls `setmetatable`, still compiles. The `manual_class` lint reports it and points to `struct` and `impl`.

**Key points**

- A `struct` declares fields and their defaults, and prints as `Name { field = value }`.
- Construct with `new Name { … }`. A missing field is a `ConstructorError` and an unknown one is a `StructError`, both before anything runs.
- An `impl` adds methods. A function without `self` is a static, and a static named `new` makes `new Name(…)` work.
- `private` and `read` emit nothing: the type checker and the `private_access` lint enforce them.
