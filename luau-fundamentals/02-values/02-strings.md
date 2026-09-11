# Strings

*Quotes, escapes, concatenation and interpolation, plus the string functions you reach for daily.*

Luau Fundamentals · Numbers, Strings & Booleans · 10 min

---

A string is text, and in Luau it is **immutable**: nothing you do to a string changes it. `string.upper(name)` does not shout at `name`, it hands you a new string and leaves the original alone. Every function in this lesson works the same way, which means the result is only useful if you keep it.

**Four ways to write one**

```luau
local a = 'single quotes'
local b = "double quotes, and \"escaped\" ones inside"
local c = [[a long string
that spans lines and needs no escapes]]

local name = "Ada"
local d = `interpolated: {name} is level {1 + 6}`

print(a)
print(b)
print(c)
print(d)
```

The last line prints `interpolated: Ada is level 7`. Single and double quotes are identical — pick whichever avoids escaping. `[[ … ]]` keeps newlines as written. Backticks interpolate, and anything inside `{}` is an expression.

## Interpolation beats concatenation

**The same sentence twice**

```luau
local name = "Ada"
local score = 42

print("The winner is " .. name .. " with " .. score .. " points")
print(`The winner is {name} with {score} points`)
```

Both print `The winner is Ada with 42 points`. Interpolation calls `tostring` on each expression, so it never raises `attempt to concatenate` — a `nil` prints as `nil` instead of stopping the script. That is a real behaviour difference, not just shorter code.

`#s` is the **length in bytes**, not in characters. For plain English text those are the same number and you will never notice. The moment a name contains an accent or an emoji they part company: `#"héllo"` is `6` because the `é` takes two bytes, while `utf8.len("héllo")` is `5`.

Use `#` when you care about storage and `utf8.len` when you are counting what a person sees.

## The string library

| Function | Signature | Does |
| --- | --- | --- |
| `string.upper` / `lower` | `(s) -> string` | An upper- or lowercased copy; the usual way to compare text case-insensitively |
| `string.sub` | `(s, i, j?) -> string` | The substring from `i` to `j`, 1-based; negative indices count from the end |
| `string.rep` | `(s, n) -> string` | `s` repeated `n` times, handy for a bar of `#` characters |
| `string.format` | `(fmt, …) -> string` | `%s` any value, `%d` a whole number, `%.2f` two decimals, `%%` a literal percent |
| `string.split` | `(s, sep?) -> { string }` | Breaks a line apart on a literal separator, default `","` |
| `string.reverse` | `(s) -> string` | Backwards, byte by byte |

Every one of these also works in **method form**: `name:upper()` is exactly `string.upper(name)`, because the string library is set up as the metatable for strings. Which form you use is a style choice; this course writes `string.upper(name)` in teaching code because it says where the function comes from, and you will see the colon form everywhere in real code.

**Key points**

- Strings are immutable: every function returns a new one, so keep the result.
- `'`, `"` and `[[ ]]` are the three literal forms; backticks add interpolation with `{}`.
- Interpolation calls `tostring`, so it never raises `attempt to concatenate`.
- `#s` counts bytes and `utf8.len(s)` counts characters; they only differ outside ASCII.
- Every `string.f(s, …)` also works as `s:f(…)`.
