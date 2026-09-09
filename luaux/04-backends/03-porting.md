# What ports, and what does not

*The markup travels between libraries; children forwarding, Instance reads and React bindings do not.*

LuauX · Backends · 11 min

---

Two lines of markup, compiled three ways. Same source, same line count, three idioms:

**The source**

```luaux
<TextLabel TextSize={16}>HP {health} / {maxHealth}</TextLabel>
<TextButton {Row} Activated={fire}>Fire</TextButton>
```

**The output**

```luau
-- React
React.createElement("TextLabel", { TextSize = 16, Text = `HP {health} / {maxHealth}` })
React.createElement("TextButton", __luaux_merge(Row, { [React.Event.Activated] = fire, Text = "Fire" }))

-- Vide
create("TextLabel")({ TextSize = 16, Text = function() return `HP {__luaux_read(health)} / {__luaux_read(maxHealth)}` end })
create("TextButton")(__luaux_merge(Row, { Activated = fire, Text = "Fire" }))

-- Fusion
scope:New("TextLabel")({ TextSize = 16, Text = scope:Computed(function(use) return `HP {use(health)} / {use(maxHealth)}` end) })
scope:New("TextButton")(__luaux_merge(Row, { [OnEvent("Activated")] = fire, Text = "Fire" }))
```

`__luaux_read` and `__luaux_merge` are one-line helpers inlined into the files that use them — which is how *no runtime* stays true without magic.

`health` is a source under Vide, a `StateObject` under Fusion and a plain value under React, and LuauX never touches which. The markup ports; the reactivity is yours and always was.

## The one place the target shows through

LuauX generates the code that **passes** children to a component. It never generates the code that **reads** them back out — that is ordinary Luau you write, and the compiler passes your expressions through untouched. So forwarding children is a line to type, not a setting to choose:

| Target | Children arrive as | Forward them with |
| --- | --- | --- |
| React | `props.children` | `{props.children}` |
| Vide | numeric keys of the props table | `{props}` |
| Fluid | numeric keys of the props table | `{props}` |
| Fusion | the key `[factory] children` names | `{props[Children]}` |

Porting a component the wrong way is loud rather than subtle — React tries to render the props table itself and fails on the first render. A spread carries children only where they are numeric keys — under Fusion they sit under `[Children]`, a string key the element's own overwrites, so a Fusion wrapper must forward them explicitly.

> **Do not interpolate a React binding**
>
> `<TextLabel>Hi {binding}</TextLabel>` builds a string, so a binding is stringified: the label reads `Hi RoactBinding(one)` and never updates. Pass it whole (`Text={binding}`) or map it (`Text={binding:map(f)}`). LuauX cannot tell a binding from a string, so this one is yours to avoid — at least it is loud, since the wrong text is on screen.

A single expression is the exception: `<TextLabel>{label}</TextLabel>` is emitted bare in every target, because a source, a `StateObject` and a string are all correct there. It is only *mixing* text with an expression that has to build a string, and building a string is what forces the wrapper.

**What you can do now**

- Say why Fusion needs no backend and React does.
- Write a `[factory]` block for any of the four targets, and know that writing one turns every default off.
- Read compiled output and recognize the two inlined helpers.
- Forward children correctly for your target, and spot the two things that do not port.
