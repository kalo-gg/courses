# Reading the output

*Open the compiled file at the same line number and read what the compiler actually emitted.*

LuauX · Best Practices & Capstone · 10 min

---

Every course so far has said the output is readable. This is the lesson that uses it. When a screen misbehaves, the fastest answer is not in the markup and not in Studio's console: it is in `build/`, on the same line number, where the compiler has written down exactly what it thought you meant. Because line N of the output is line N of the source, finding it needs no map — open the twin file and go to the line the trace names.

**Line 7, before and after**

```luau
-- src/client/Hud.luaux, line 7
<TextLabel Text={"Score: " .. score()} />

-- build/client/Hud.luau, line 7
create("TextLabel")({ Text = "Score: " .. score() })
```

There is the bug, spelled out: `Text` is a string, computed once. Nothing in the markup looked wrong, and nothing in the output is hiding.

Two names appear in the output that you never wrote. `__luaux_read(x)` calls `x` if it is a function and returns it otherwise; it appears inside a wrapped interpolation under the table arrangement. `__luaux_merge(a, b)` is the spread helper, and it appears wherever you wrote a spread unless `[factory] merge` names your library's own. Both are one-liners inlined into the files that use them — the whole of what "no runtime" costs.

## Five shapes worth recognizing

| In the output | What it means |
| --- | --- |
| `Text = "…" .. x()` | The attribute form: read once, frozen |
| `Text = function() return … end` | Interpolated text, wrapped by the compiler |
| `Text = x` | The source passed whole; the library re-reads it |
| `React.createElement(…)` in a Vide project | No `[factory]` block is reaching the compiler |
| `__luaux_merge(a, { … })` | A spread; whatever is in the second table wins |

Five readings, and each one answers a different question from the checklist. The output is the only place all five are visible at once.

**The output of a wrong block**

```luau
-- build/client/Row.luau — what this project is getting
scope:New("Frame")(__luaux_merge(Styles.Row, { [OnEvent("Activated")] = props.OnClick, [Children] = { child } }))

-- what a Vide project needs
create("Frame")(__luaux_merge(Styles.Row, { Activated = props.OnClick, child }))
```

Same markup, same arrangement, wrong contents: a constructor that does not exist here, an event key that is a call, and children moved under a name Vide does not use.

Three things in that first line are wrong and none of them is in the markup: a constructor this project does not have, an event key that calls, and children under a name Vide never reads. The emitted call is evidence of what the **config** said — and a whole class of bug is a `[factory]` block copied from a project targeting something else.

**Key points**

- The compiled twin is the debugger: same file name, same line number.
- `__luaux_read` marks a wrapped interpolation; `__luaux_merge` marks a spread.
- Five shapes in the output answer five questions from the checklist.
- Fix the source and rebuild; an edit to `build/` is gone at the next `clean`.
