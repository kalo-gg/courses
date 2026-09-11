# Finding text

*find, match, gsub and gmatch, and the pattern characters that make them worth learning.*

Luau Fundamentals · Numbers, Strings & Booleans · 10 min

---

Four functions cover nearly all the text work you will ever do, and they share one idea: a **pattern**, a small language for describing the shape of the text you are looking for. Learn the pattern once and all four become available at the same time.

| Function | Answers | Returns |
| --- | --- | --- |
| `string.find(s, pat)` | *Where* is it? | The start and end indices, or `nil` |
| `string.match(s, pat)` | *What* is it? | The captures, or the whole match, or `nil` |
| `string.gsub(s, pat, new)` | Replace them all | The new string **and** the number of replacements |
| `string.gmatch(s, pat)` | Give me each one | An iterator, for a generic `for` |

## Patterns are not regular expressions

Lua patterns look enough like regular expressions to fool you and are a different, smaller language. The classes are `%d` a digit, `%a` a letter, `%s` whitespace, `%w` a letter or digit, `%p` punctuation, and `.` for any character at all. The repeats are `+` one or more, `*` zero or more, `-` as few as possible, and `?` optional.

The differences that catch people out: **`%` is the escape character, not `\`** — a literal dot is `%.`, not `\.`. There is **no alternation**: no `|`, and no `(a|b)` group. And the classes are one character each, so `%d+` is fine but `\d{2,4}` means nothing here.

**One pattern, four functions**

```luau
local line = "go north 3 times"

print(string.find(line, "%d+"))  -- 10  10
print(string.match(line, "%d+")) -- 3
print(string.gsub(line, "%d+", "N")) -- go north N times   1

for word in string.gmatch(line, "%a+") do
	print("word", word)
end
```

The `for` prints `go`, `north` and `times` — `%a+` skips the digits because a digit is not a letter. `find` gives positions, `match` gives text, `gsub` rewrites, `gmatch` visits.

> **`gsub` returns two values**
>
> This is the first place in the course where a function returns more than one value, and it will not be the last: Luau's standard library does it constantly. `local s = string.gsub(…)` quietly keeps the first and drops the count, which is usually what you want. `print(string.gsub(…))` keeps both, which is almost never what you want. Functions & Scope has a whole lesson on the rule underneath this.

When the thing you are searching for contains a pattern character, you have two ways out. Escape it with `%` — `string.match(version, "%d+%.%d+")` finds `0.733` — or, for `find` only, pass `true` as the fourth argument to search for a plain string instead of a pattern.

**A dot is not a dot**

```luau
print(string.find("a.b", "."))       -- 1  1: "." matches ANY character
print(string.find("a.b", "%."))      -- 2  2: "%." matches a literal dot
print(string.find("a.b", ".", 1, true)) -- 2  2: plain search, no pattern

print(string.match("0.733", "%d+%.%d+")) -- 0.733
```

The first line is the bug: `.` matched the very first character, `a`. Nearly every "my pattern matches too much" report is a `.`, a `-` or a `%` that should have been escaped.

**Key points**

- `find` locates, `match` extracts, `gsub` rewrites, `gmatch` iterates — one pattern language behind all four.
- Lua patterns escape with `%`, not `\`, and have no alternation.
- `%a` letters, `%d` digits, `%s` whitespace, `%w` both letters and digits, `.` anything.
- `gsub` returns the string **and** a count; wrap it in parentheses to keep only the string.
