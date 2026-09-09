# Renaming the vocabulary

*Give a project its own names for classes and properties, and learn the two rules that make a rename safe.*

LuauX · The Syntax · 10 min

---

Some projects would rather write `<text>` than `<TextLabel>`, or `bgColor` than `BackgroundColor3`. LuauX allows it, in `luaux.toml`, with two rules attached. The keys you rename **from** are spelled the way Roblox spells them, because those names are borrowed rather than invented. And a rename is **exclusive**: once a new spelling exists, the old one stops being valid. One vocabulary per project, never two.

A project can rename elements and properties in `luaux.toml` — one at a time, or wholesale with `all`. The keys use **canonical Roblox spelling**, not TOML's usual snake_case, because those names are borrowed from the Roblox API rather than invented by LuauX.

**Renaming**

```toml
[elements]
TextLabel = "text"      # <text> in your markup
all = "flatcase"        # PascalCase | camelCase | snake_case | flatcase

[properties]
all = "camelCase"

[properties.Frame]
BackgroundColor3 = "bgColor"   # or for one class only
```

Everything LuauX defines itself stays snake_case; only these two sections borrow Roblox's spelling. An explicit entry always beats `all`.

> **Renames are exclusive**
>
> Once `TextLabel = "text"` is declared, `<text>` is *the* spelling and `<TextLabel>` is an error. `all` behaves the same way, retiring every canonical spelling at once. One vocabulary in a project, not two — which is the only thing that makes a rename worth having.

## Which spelling goes where

`all` applies a scheme to every name at once — `PascalCase`, `camelCase`, `snake_case` or `flatcase` — and an explicit entry always beats it, so a project can be camelCase everywhere with one exception spelled out. `[properties.Frame]` scopes a rename to one class. Everything LuauX defines itself (`in`, `out`, `clean`, the lint names) stays snake_case; only these two sections borrow Roblox's spelling.

An alias is also the first thing a tag name is checked against — before the class list, before anything bound in the file. The Components module walks the other two steps.

**The same Frame, in the new vocabulary**

```luaux
<!-- before: every name spelled the way Roblox spells it -->
<Frame BackgroundColor3={panel} BorderSizePixel={0} />

<!-- after: [properties] all = "camelCase", and no [elements] entry for Frame -->
<Frame backgroundColor3={panel} borderSizePixel={0} />
```

`Frame` has no entry in `[elements]`, so it keeps Roblox's spelling; both of its properties are caught by `all`. Rename what the project says out loud and leave the rest alone — and remember that after this config the first line no longer compiles. The second exercise asks you to do the same to a `TextLabel`, where an explicit entry is in play as well.

**Key points**

- `[elements]` and `[properties]` rename; `all` applies a scheme; an explicit entry beats it.
- Keys use canonical Roblox spelling; LuauX's own keys are snake_case.
- Renames are exclusive: the old spelling becomes an error.
- An alias is the first step of tag resolution.
