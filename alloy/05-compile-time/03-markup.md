# Markup

*`.alx` files: tags that lower to your UI library, typed attributes, and conditional children that re-run.*

Alloy · Compile-time Alloy · 10 min

---

A shop row built in plain Luau is a nest of calls, `React.createElement('TextButton', { … }, …)` inside another. An **`.alx` file** is Alloy with markup in it, the way JSX sits in TypeScript. Tags lower to calls on your UI library, and the Alloy around them compiles as in any `.aly` file.

## A component in an .alx file

**Row.alx**

```alx
import * as React from '@game/ReplicatedStorage/Packages/React'

type RowProps = {
  name: string,
  on_pick: () -> (),
}

local function Row(props: RowProps)
  return (
    <TextButton Size={new UDim2(1, 0, 0, 24)} Activated={props.on_pick}>
      {props.name}
    </TextButton>
  )
end

return Row
```

**Row.luau, with no [alx] table**

```luau
local React = require('@game/ReplicatedStorage/Packages/React')

type RowProps = {
  name: string,
  on_pick: () -> (),
}

local function Row(props: RowProps)
  return (
    React.createElement("TextButton", { Size = UDim2.new(1, 0, 0, 24), [React.Event.Activated] = props.on_pick,
      Text = props.name }
    )
  )
end

return Row
```

Line for line, as in any `.aly` file. The event becomes `[React.Event.Activated]`, and since `TextButton` has a `Text` property, the lone `{props.name}` between the tags sets it.

A tag is a Roblox class or a component bound in the file; any other name is a compile error with a suggestion. An attribute is `Name={expr}` or `Name="text"`, typed against the property or against the prop the component declares. Declare props as a `type` or an `interface`: a `struct` is nominal, so no tag can satisfy it. A bare `{props}` spreads a table into the attributes, `<> … </>` is a fragment, and `<!-- -->` is a comment.

**alloy.toml, for Vide**

```toml
[alx.factory]
backend = "table"
create = "create"
```

With this table, `<TextLabel Text={props.name} />` lowers to `create("TextLabel")({ Text = props.name })`, and `create` must be in scope. `children`, `event`, `compute`, `interpolate` and `merge` name the rest of your library's shape.

## Children that re-run

Under the table form, a child expression runs once, when the tag is built. So `{if open then <Menu /> else nil}` keeps its first value, and when `open` changes, nothing shows. Wrap it in a function, `{function() return if open then <Menu /> else nil end}`, and Vide calls it again. The `alx.static_conditional_child` lint warns about the bare form there. Under React it is off: a React component renders again by itself, so the bare form is right.

> **Recognising older code**
>
> Older projects keep markup settings in a `luaux.toml` beside the file and set the lint with `[alx.lints] static_conditional_child`. Today both live in `alloy.toml`: the lowering in `[alx]`, and the level in `[lint.rules]` as `alx.static_conditional_child`. A `luaux.toml` is still read while `[alx]` is empty.

> **Checked as text here**
>
> This course's sandbox cannot compile `.alx` files, so the exercise below is checked as text and does not run. Kalo's LuauX course covers markup and its UI libraries in depth: Alloy's markup is a fork of luaux.

**Key points**

- An `.alx` file is Alloy with markup; a tag lowers to a call on your UI library.
- With no `[alx]`, markup lowers to `React.createElement`; `[alx.factory] backend = "table"` selects the table form for Vide or Fusion.
- Attributes are typed against the property, or the prop a component declares as a `type` or `interface`.
- Under the table form, a conditional child is built once unless a function wraps it; `alx.static_conditional_child` warns there and is off under React.
