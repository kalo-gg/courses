# Text and interpolation

*Text between tags, the one construct that becomes reactive on its own, and the attribute that never does.*

LuauX · The Syntax · 10 min

---

Text between tags becomes the element's text. Braces inside that text interpolate an expression.

**Interpolated text**

```luaux
<TextLabel>Clicked {clicks} times</TextLabel>
```

> **This is the only implicit reactivity in LuauX**
>
> Interpolated text is the one construct the compiler wraps for you, because building a string out of a reactive value would otherwise stringify the value itself. Nowhere else does LuauX add reactivity: an attribute takes exactly the value you wrote, and if you want it to react you pass a function.

> **What the wrapper is depends on your target**
>
> This course's examples target **Vide**, where the wrapper is a thunk — `Text = function() … end` — so the label tracks `clicks`. Under Fusion it is a `scope:Computed`, and under **React, which is what LuauX targets when you write no `[factory]` block at all**, there is no wrapper: the interpolation is a plain string, and a React binding put there would be stringified. The Backends module is where this stops being a footnote.

## The mistake worth remembering

**The difference, side by side**

```luaux
<!-- reactive: the compiler wraps the text in a thunk -->
<TextLabel>Clicked {clicks} times</TextLabel>

<!-- NOT reactive: this reads clicks once, at build time -->
<TextLabel Text={"Clicked " .. clicks() .. " times"} />

<!-- reactive again, because you wrote the function -->
<TextLabel Text={function() return "Clicked " .. clicks() .. " times" end} />
```

The middle one is the mistake worth remembering: it compiles, runs, and never updates.

Inside the tags either spelling works — `{score}` or `{score()}` — because the whole template runs inside the wrapper, so a read in there is a read the library sees. It is the **attribute** form that reads once: `Text={"Score: " .. score()}` builds its string before any wrapper exists, and the label keeps whatever the score was at that moment.

One exception is worth knowing now: a child that is a *single* expression with no text around it — `<TextLabel>{label}</TextLabel>` — is emitted bare in every target, because a source, a `StateObject` and a string are each already right there. Only mixing text with an expression has to build a string, and building a string is what forces the wrapper.

**Key points**

- Text between tags is the element's text; braces inside it interpolate.
- Interpolated text is the only construct LuauX wraps for you; under Vide the wrapper is a thunk.
- An attribute takes exactly the value you wrote — pass a function if it should react.
- A single expression child is emitted bare; only mixed text builds a string.
