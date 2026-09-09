# Strings, collections and types

*toContain, toHaveLength, toMatch and toBeA, and the one that takes a pattern rather than a substring.*

Lest · Matchers · 10 min

---

A test about a string or a list usually wants to say *contains this*, *has this shape* or *is this long*. Four matchers cover it, and they follow Luau rather than JavaScript: `#` is the array length, patterns are Luau patterns, and `typeof` is what decides a type.

| Matcher | Meaning |
| --- | --- |
| `toContain(item)` | For a **string**, a plain substring. For an **array**, any element **deeply equal** to `item`. |
| `toHaveLength(n)` | `#value`, for strings and tables — so **the array part only**. |
| `toMatch(pattern)` | A **Luau string pattern** — not a substring, and not a regex. |
| `toBeA(typeName)` | A `typeof` check, so it knows `'Instance'`, `'Vector3'` and `'CFrame'` too. |

**Collections**

```luau
expect('hello world').toContain('lo wo')
expect({ 'a', 'b' }).toContain('b')
expect({ { id = 1 } }).toContain({ id = 1 })   -- deep, not by reference

expect('abc').toHaveLength(3)
expect(id).toMatch('^user_%d+$')
```

`toContain` on an array is deep because reference equality is rarely what a Luau test means.

> **`toHaveLength` on a dictionary is always 0**
>
> `#` is Luau's array length, and a table with only string keys has none. `expect({ a = 1, b = 2 }).toHaveLength(2)` does not throw and does not pass — it fails with *"expected … to have length 2, but its length is 0"*, which reads like the code is wrong when the assertion is. Jest's `.toHaveLength` reads `.length` and throws on a plain object, so this is one more place the Luau spelling means something else. Count a dictionary yourself, or assert on it with `toEqual`.

> **`toMatch` is a pattern, `toContain` is a literal**
>
> This is the easiest mistake to make. `toMatch('a.b')` matches `axb`, because `.` is a pattern metacharacter. When you want a literal string, reach for `toContain`.

The choice between those two is the one that bites: `toContain` is a literal, `toMatch` is a pattern, and a dot, a dash or a percent sign in your string means something to a pattern. When the thing you are checking for is a fixed piece of text, `toContain`; when it is a *shape*, `toMatch`.

## Types

`toBeA` compares against `typeof`, so `'string'`, `'number'`, `'table'` and `'function'` work everywhere, and `'Instance'`, `'Vector3'` and `'CFrame'` work on the engine backends. It reads best as the first link in a chain — *this is a string, and it looks like this* — which the next lesson covers.

**A typeof check**

```luau
expect(cart.total).toBeA('function')
expect(cart.total({ 1 })).toBeA('number')
expect(workspace).toBeA('Instance')        -- on cloud or studio
```

**Key points**

- `toContain`: substring of a string, or an element deeply equal to the item in an array.
- `toHaveLength` is `#`: strings and arrays, never dictionaries.
- `toMatch` takes a Luau pattern; use `toContain` for a literal.
- `toBeA` is a `typeof` check and knows Roblox types on the engine backends.
