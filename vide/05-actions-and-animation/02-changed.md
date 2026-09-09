# Listening with changed

*Turn a property the engine changes into a source with the changed action, and bind both ways.*

Vide · Actions, Springs & Context · 9 min

---

Bindings push state *into* instances. `changed(property, fn)` goes the other way: it is an action that calls `fn` with the property's value immediately and again every time the property changes. Pass a source as `fn` and the property becomes readable state, which is how you get text out of a `TextBox` or the size of an auto-sized frame.

```luau
-- inside a root or component
local changed = vide.changed

local text = source("")

create "TextBox" {
	changed("Text", text),
}

effect(function()
	print("typed:", text())
end)
-- "typed: " immediately; then once per edit
```

`changed` is an action, so it goes in an unkeyed slot.

Under the hood `changed` is `GetPropertyChangedSignal(property):Connect(…)` followed by `cleanup` of the connection, then one immediate call with the current value. That `cleanup` is why it needs a scope: outside `root` or `mount` the error is `cannot cleanup outside a stable or reactive scope`, and inside a dynamic scope the connection dies with the row or the menu.

## Two-way binding

Combining both directions gives a two-way binding: bind `Text` to a source with an implicit effect, and feed edits back with `changed`. Because writing an equal value is a no-op, the two do not fight.

**Two-way text**

```luau
local name = source("Ada")

create "TextBox" {
	Text = name,            -- source in
	changed("Text", name),  -- edits out
}
```

**Key points**

- `changed(property, fn)` calls `fn` with the value now and on every change; pass a source to capture it.
- Combine `Text = src` with `changed("Text", src)` for two-way binding.
- The connection is cleaned up with the scope automatically.
