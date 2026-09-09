# Elements and attributes

*A Roblox class name, some attributes, children between the tags, and two shorthands that say it once.*

LuauX · The Syntax · 10 min

---

Every screen from here is a few hundred tags, and each one is a class name, some attributes and some children — so this is the lesson the rest of the course sits on. An element is a Roblox class name in angle brackets. Its attributes are the properties you would have written in a table, its children sit between the opening and closing tags, and an element with no children closes itself. That is nearly all of the syntax; the rest of this lesson is the two places it saves you typing.

**The basic shape**

```luaux
<TextButton
	Size={UDim2.fromOffset(160, 40)}
	Text="Click me"
	BackgroundColor3={Color3.fromRGB(30, 30, 30)}
>
	<UICorner CornerRadius={UDim.new(0, 8)} />
</TextButton>
```

A tag name is a Roblox class. An attribute is `Name="literal"` or `Attr={expression}`.

Braces hold any Luau expression, so `{UDim2.fromOffset(160, 40)}` is just a call. A quoted string needs no braces, which keeps the common case short. An element with no children closes itself: `<UICorner … />`.

## Two shorthands

**Saying it once**

```luaux
<!-- A bare name is = true -->
<TextLabel TextWrapped />

<!-- When the expression already says which property it is -->
<Frame ={props.Size} ={props.Visible} />
```

The second compiles to `Size = props.Size, Visible = props.Visible`.

The inferred form has one rule: the expression has to **be** a name — an identifier, or a dotted path whose last segment is the property. `={f().Size}` or `={a or b}` is an error telling you to write the property out, because a shorthand you cannot predict is worse than one you type in full. It is spelled `=` rather than a bare `{Size}` because a bare hole in attribute position already means a spread.

> **The compiler knows the classes**
>
> Tag names and attribute names are checked against the real Roblox API, so a misspelled `TextLable` or a `Size` on something that has none is caught at compile time rather than at play time.

**Key points**

- A tag is a Roblox class; an attribute is `Name="literal"` or `Name={expression}`.
- A bare attribute name means `true`; `={path.Prop}` names the property from the path.
- The inferred form needs a name, not a computation — `={a or b}` is an error.
- Tags and attributes are checked against the real Roblox API at compile time.
