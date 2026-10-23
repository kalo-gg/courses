# Enums

*List a type's cases with `enum`: unit variants that are plain strings, payload variants that carry data, and methods called on the enum.*

Alloy · Structs and enums · 10 min

---

Roblox code spells states as strings: `'Idle'`, `'Busy'`, and one day `'Buisy'`, which no tool catches. Messages are worse: a table with a `kind` field, plus whatever other fields each sender remembered to set.

An **enum** lists the cases once. Each case is a *variant*, and a variant can carry a payload.

## Unit enums are strings

**status.aly**

```alloy
enum Status
  Idle
  Busy
end

impl Status
  function describe(self) -> string
    return self == Status.Idle ? 'waiting' : 'working'
  end
end

const s = Status.Busy
print(s == 'Busy', Status.describe(s))
```

This prints `true working`. `Status.Busy` *is* the string `'Busy'`, so it compares, stores and prints like one.

**status.luau (ship)**

```luau
local __alloy = require("@alloy") local Status = {} Status.__index = Status
Status.Idle = "Idle"
  Status.Busy = "Busy"
function Status.is(v) return (type(v) == "table" and getmetatable(v) == Status) or v == "Idle" or v == "Busy" end Status.__tostring = function(v) return __alloy.show_variant("Status", v) end type Status = "Idle" | "Busy"

do
  function Status.describe(self): string
    return (if self == Status.Idle then 'waiting' else 'working')
  end
end

const s = Status.Busy
print(s == 'Busy', Status.describe(s)) return nil
```

Each variant becomes a string on a table, on its own line. `Status.is` tests a value, and the type is the union `"Idle" | "Busy"`.

## Payload enums

A variant with a payload is a constructor. `Msg.Chat('hi')` builds a table tagged `"Chat"`, with the payload in `_1`, `_2` and so on. Unit variants in the same enum stay strings, so `Msg.Quit` is `'Quit'`. A payload value prints with its enum's name and its variant's name.

**msg.aly**

```alloy
enum Msg
  Chat(string)
  Kick(string, number)
  Quit
end

const m = Msg.Chat('hi')
print(m, m.tag, m._1)
print(Msg.Quit, Msg.is(Msg.Quit))
```

This prints `Msg.Chat("hi")  Chat  hi`, then `Quit  true`. You rarely read `tag` or `_1` yourself: the next lesson's `match` reads them for you.

**msg.luau (ship)**

```luau
local __alloy = require("@alloy") local Msg = {} Msg.__index = Msg
function Msg.Chat(_1) return setmetatable({ tag = "Chat", _1 = _1 }, Msg) end
  function Msg.Kick(_1, _2) return setmetatable({ tag = "Kick", _1 = _1, _2 = _2 }, Msg) end
  Msg.Quit = "Quit"
function Msg.is(v) return (type(v) == "table" and getmetatable(v) == Msg) or v == "Quit" end Msg.__tostring = function(v) return __alloy.show_variant("Msg", v) end type Msg = typeof(setmetatable({} :: { tag: "Chat", _1: string }, Msg)) | typeof(setmetatable({} :: { tag: "Kick", _1: string, _2: number }, Msg)) | "Quit"

const m = Msg.Chat('hi')
print(m, m.tag, m._1)
print(Msg.Quit, Msg.is(Msg.Quit)) return nil
```

There is one constructor per payload variant, each on its variant's line. The type is a union of tagged tables and `"Quit"`.

`Msg.is(v)` tests whether a value belongs to the enum. An enum can take type parameters: in `enum Slot<T>`, the variant `Full(T)` carries a `T`, and `impl Slot<T>` names `T` in a method. An `impl` works on any enum. A colon call works on a payload value, which has the enum as its metatable, but not on a unit value.

**Key points**

- A unit enum is a string union at run time: `Status.Busy == 'Busy'`.
- A payload variant is a constructor that builds a tagged table, and it prints as `Msg.Chat("hi")`.
- `E.is(v)` tests whether a value belongs to the enum, and `enum Name<T>` takes type parameters.
- Call a unit enum's methods on the enum, as in `Status.describe(s)`, because a unit value is just a string.
