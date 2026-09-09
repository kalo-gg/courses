# What the compiler catches

*Tags checked against 356 classes, attributes against 899, a lint with three levels, and what is still yours to avoid.*

LuauX · Tooling & Adoption · 10 min

---

The most valuable thing LuauX does is not syntax. It is that **it knows Roblox**: tags are checked against the 356 creatable classes, attributes against the inheritance chain of 899. A mistake surfaces before Rojo syncs rather than as a silent no-op in Studio.

**A typo fails the build**

```text
  × Frame has no property or event named BackgroundColour3
   ╭─[src/App.luaux:7:5]
 7 │     BackgroundColour3={Color3.new()}
   ·     ────────┬────────
   ·             ╰── did you mean BackgroundColor3?
   ╰────
```

The location is real because the diagnostic points at your source. Lines are preserved too — line N of the output is line N of the source — so a Studio stack trace on `build/App.luau:42` is about `src/App.luaux:42`. Columns are not: the inlined helpers sit at the head of line 1.

That last point is worth dwelling on. Every construct lands on the line it came from, so a stack trace on `build/App.luau:42` really is about `src/App.luaux:42`. A compiled language you cannot debug is a bad trade; this is the property that makes the trade a good one.

## Lints

**luaux.toml**

```toml
[lints]
static_conditional_child = "warn"
```

The lesson "Children that change" covered this one: a conditional child built once, so the UI never updates.

The compiler's own lint has three levels — `off`, `warn` and `error` — and its default follows the backend: `warn` under `table`, `off` under `element`. Set it to `error` and `luaux check` exits non-zero when the pattern appears, which turns the lint into a gate on every pull request. This is LuauX's lint, configured in `luaux.toml` and applied when *you* run the compiler; the next lesson shows the other lint scale — the worm's own rules, levelled in `larvae.toml`, which the compiler never reports and `luaux check` never sees.

The boundary is worth drawing precisely. A wrong class or property name and a `[factory]` name that is not in scope fail the compile. A conditional child built once is a lint. A React binding mixed into text, or a source replaced by a plain field, is ordinary Luau the compiler cannot see — those stay yours.

**Key points**

- Tags and attributes are checked against Roblox's real API; a typo fails the build with a suggestion.
- Diagnostics and stack traces both point at the source line, because lines are preserved.
- `[lints] static_conditional_child` is `off`, `warn` or `error`; `error` makes `luaux check` a CI gate.
- Runtime meaning — bindings, sources, Instances — is the library's; the compiler cannot see it.
