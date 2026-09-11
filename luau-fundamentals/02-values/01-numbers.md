# Numbers and arithmetic

*Every number is a double, and Luau adds floor division, compound assignment and math helpers.*

Luau Fundamentals · Numbers, Strings & Booleans · 10 min

---

Luau has exactly **one** number type: a 64-bit floating point number, the same thing most languages call a `double`. There is no separate integer. A whole number is just a number with nothing after the point, which is why `1 / 2` is `0.5` and not `0`, and why you never have to convert between two numeric kinds.

That one type is accurate for whole numbers up to about nine quadrillion, so counting things is never a problem. Fractions are a different story, and `0.1 + 0.2` prints `0.30000000000000004` here exactly as it does in JavaScript or Python.

**Arithmetic**

```luau
print(7 + 2, 7 - 2, 7 * 2)
print(7 / 2)  -- 3.5, because division always gives a float
print(7 // 2) -- 3, floor division: divide, then round down
print(7 % 2)  -- 1, the remainder left over
print(2 ^ 10) -- 1024, exponentiation
```

`//` and `%` are a pair: `a // b` is how many whole times `b` fits, and `a % b` is what is left. Together they turn 197 seconds into 3 minutes and 17 seconds.

## Compound assignment

**Shorter, and the same**

```luau
local health = 100
health -= 35
health -= 35
print(health) -- 30

local coins = 3
coins *= 2
print(coins) -- 6

local line = "go"
line ..= " north"
print(line) -- go north
```

`+=`, `-=`, `*=`, `/=`, `//=`, `%=`, `^=` and `..=` all exist. Each one names its variable once, so there is one fewer place to mistype it. Lua 5.1 has none of them and writes `health = health - 35`.

Precedence is the usual one — `^` first, then `*` `/` `//` `%`, then `+` `-`, then `..` — with one rule worth memorising because it surprises everybody: **`^` binds tighter than unary minus.** `-2 ^ 2` is `-4`, because it means `-(2 ^ 2)`. If you meant the other thing, write `(-2) ^ 2`.

## The math library

| Function | Gives | Reach for it when |
| --- | --- | --- |
| `math.floor(x)` | Rounds down: `2.7` → `2`, `-1.5` → `-2` | You need a whole number and down is the safe direction |
| `math.ceil(x)` | Rounds up: `2.1` → `3` | You are counting pages or rows and a partial one still costs a whole one |
| `math.round(x)` | Nearest, halves away from zero: `2.5` → `3` | You are showing a number to a person |
| `math.abs(x)` | Distance from zero | You care how far apart two values are, not which is bigger |
| `math.min(…)` / `math.max(…)` | The smallest / largest argument | `math.max(health - damage, 0)` keeps health off the floor |
| `math.clamp(x, lo, hi)` | `x` pinned into a range | A slider, a volume, a percentage |
| `math.random(m, n)` | A whole number in `[m, n]` | Loot tables and spawn points |

`math.round` is one of Luau's additions. Lua 5.1 code fakes it with `math.floor(x + 0.5)`, which is subtly wrong for negative numbers — `math.floor(-2.5 + 0.5)` is `-2`, where `math.round(-2.5)` is `-3`. If you find that idiom in a tutorial, it is a Lua tutorial.

One more worth knowing about: `math.noise` is part of core Luau, not something Roblox added. It is the most commonly misattributed function in the whole library.

**Key points**

- One number type, a 64-bit float, so `7 / 2` is `3.5` and there is nothing to convert.
- `//` rounds down and `%` takes the sign of the divisor; both surprise you first with a negative.
- `+=`, `-=`, `*=` and `..=` are Luau's, and name the variable once.
- `^` binds tighter than unary minus: `-2 ^ 2` is `-4`.
