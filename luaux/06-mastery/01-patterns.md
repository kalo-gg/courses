# Patterns that scale

*Style tables, dotted namespaces and props that take a value or a source: the habits a screen needs.*

LuauX · Best Practices & Capstone · 10 min

---

Twenty components in, nothing about LuauX has changed and everything about the repository has. The same six attributes appear on every card, two components are one Roblox class name away from being unreachable, and a source someone created inside a component cannot be reset from outside it. None of this is a compiler problem, which is exactly why the compiler will never warn you about any of it.

**src/client/Styles.luau, and a file that uses it**

```luaux
-- Styles.luau — plain Luau, no markup, copied into the output by the build
Styles.Card = {
	BackgroundColor3 = Color3.fromRGB(28, 28, 32),
	BorderSizePixel = 0,
	Size = UDim2.new(1, 0, 0, 96),
}

-- Card.luaux — one spread, and the caller can still override anything after it
<Frame {Styles.Card} {props} />
```

A style table is a spread, not a component: it adds no element, no scope and no file to the tree, and the theme changes in one place.

Why a table and not a `<Card>` wrapper? Because a spread is not an element: `{Styles.Card}` merges into the props table the constructor was going to receive anyway, and anything written after it still wins. A wrapper would add a Frame nobody asked for, and a layout bug two months later.

**src/client/UI.luau — one namespace for shared components**

```luau
local UI = {
	Card = require(script.Parent.Card),
	Dialog = require(script.Parent.ConfirmDialog),
	Row = require(script.Parent.Row),
}

return UI
```

Another file with no markup, so the build copies it. Importers write `<UI.Card />` and stop thinking about Roblox's class list.

## Props that take either

Two habits left, both about who owns a value. Write props that accept a **value or a source** — `Label: string | vide.Source<string>`, the type spelled as the module member Vide exports — and pass them straight to the attribute: the library reads a function on a property key as reactive and a plain value as an assignment, so neither case needs handling. (`vide.read(value)` is there for the rare component that must look at the value itself.) And keep every source in the file that assembles the screen: a component that creates its own source is one nothing outside it can reset.

| Habit | Instead of | Why |
| --- | --- | --- |
| A shared prop table, spread | The same six attributes on every card | The theme changes in one file, and the spread is inlined |
| `<UI.Card />` | `<Card />`, and hoping | A dotted name cannot collide with a Roblox class |
| `Label: string \| vide.Source<string>` | Two components, or a `read` in every attribute | The library already tells the two apart |
| Sources in `App`, passed down | A source created inside the component | State you can reset, replay and hand to two screens |

None of these is enforced. All four are the difference between a screen that survives a year and one that gets rewritten.

**Key points**

- Shared attributes belong in a plain `.luau` table the build copies, spread into elements.
- A dotted name (`<UI.Card />`) can never collide with a Roblox class; a bare one can.
- Props that take a value or a source cost nothing: pass them straight to the attribute.
- Sources live in the file that assembles the screen, not inside the components that display them.
