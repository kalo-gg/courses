# Macros and intrinsics

*Templates pasted at each `$name(…)` call, their hygiene and limits, and the closed set of built-in intrinsics.*

Alloy · Compile-time Alloy · 10 min

---

Every health bar and tween clamps to the 0–1 range: `math.clamp(x, 0, 1)`, again and again. A helper function gives it a name, but adds a call on every frame. And an assert helper can never print the expression it was given, because a function only ever sees values.

A **macro** works on source instead. The compiler pastes its body in at each `$name(…)` call, before anything runs.

## Declare it, call it with $

```alloy
macro clamp01(x)
  math.clamp(x, 0, 1)
end

const health = $clamp01(1.4)
const ratio = 10 / $clamp01(0.5 + 0.25)
print(health, ratio)
```

**Emitted Luau**

```luau

const health = math.clamp(1.4, 0, 1)
const ratio = 10 / math.clamp((0.5 + 0.25), 0, 1)
print(health, ratio) return nil
```

The macro's lines go blank, so every later line keeps its number. An argument of more than one term lands in parentheses, so `0.5 + 0.25` stays whole. A file with no `return` ends in ` return nil`.

Expansion is textual but hygienic: every `local` in the body is renamed for each expansion, so it never collides with yours. A parameter can take a default, `macro sum(a, b = 2)`, and `...` makes a macro variadic. The body sees its parameters and whatever is in scope where it lands. An `export macro` does not carry its file's locals along, so pass values in as parameters. A macro cannot call itself: that is a `MacroError`.

```alloy
macro log_twice(msg)
  const text = `[log] {msg}`
  print(text)
  print(text)
end

const text = 'saved'
$log_twice(text)
```

The call expands to `const text__m1 = \`[log] {text}\` print(text__m1) print(text__m1)`, all on line 8. The body's `text` is renamed, so it cannot shadow yours.

## The intrinsics

Some `$` names are built in. These **intrinsics** are a closed set that you cannot add to, and several of them can see your source text, which no function can.

| Intrinsic | Becomes |
| --- | --- |
| `$assert(cond)` | `assert(cond, "assertion failed: <source>")` |
| `$assert_eq(a, b)` | An error naming both source texts and both values when they differ |
| `$dbg(x)` | Prints `file:line: <source> = <value>` and returns `x` |
| `$todo('why')`, `$unreachable()` | An error carrying the file and line |
| `$nameof(a.b.c)`, `$stringify(expr)` | The string `"c"`; the expression's source text |
| `$set[1, 2]`, `$map[['a', 1]]` | A `Set` or a `HashMap` of the values |
| `$matches(v, pattern)` | `true` when `v` fits the pattern, as a `case` would |
| `$bnot(x)` | `bit32.bnot(x)` |
| `$expect(v)` | Lest's matchers, inside a `@test` only (lesson 4) |

**Key points**

- `macro name(params) … end` declares a template; `$name(…)` pastes it in at compile time.
- The declaration's lines go blank, and a many-term argument lands in parentheses.
- Locals in the body are renamed per expansion; parameters take defaults and `...`.
- A macro cannot recurse (`MacroError`), and an exported macro carries no locals of its file.
- The intrinsics are a closed set: `$assert`, `$assert_eq`, `$dbg`, `$nameof`, `$stringify`, `$set`, `$map`, `$matches` and more.
