# Children that change

*A conditional or a list built once never updates; the lint that catches it and the helpers to reach for.*

LuauX · Components · 11 min

---

A hole in child position takes any Luau expression, and whatever it evaluates to is handed to your library. That is the whole mechanism — which means a list or a conditional is built out of your library's own helpers, not out of LuauX syntax.

**A conditional child, written twice**

```luaux
<!-- Wrong. This evaluates once, when the Frame is built. -->
{#todos() == 0 and <TextLabel>Nothing to do</TextLabel> or nil}

<!-- Right. show() re-evaluates and swaps the child. -->
{show(function()
	return #todos() == 0
end, function()
	return (<TextLabel TextTransparency={0.5}>Nothing to do</TextLabel>)
end)}
```

From `examples/TodoList.luaux`. Both compile. Only one updates.

> **`static_conditional_child`**
>
> LuauX ships a lint for the first form, because it is the mistake worth catching: **a conditional child that the code builds one time, so the UI does not update.** Nothing is wrong with the Luau — `and`/`or` runs once, as it always has — and nothing is wrong with the markup. It simply never runs again.

## Lists

The same reasoning covers lists. `indexes` keeps one instance per position and updates it in place, so editing an entry does not rebuild its row; it returns a source of children, which the library recurses.

Written as a plain loop in a hole, it would build the rows once and stop there: the same mistake as the conditional above, one construct larger. Nothing warns about this one, because a `for` loop in a hole is legal Luau doing exactly what it says. The symptom is what tells you — a list that is right on the first frame and never right again.

**A dynamic list**

```luaux
{indexes(todos, function(todo, index)
	return Row(todo, function() toggle(index) end)
end)}
```

> **The lint's default depends on your backend**
>
> Under the **table** arrangement — Vide, Fluid, Fusion — it defaults to `warn`, because such a child really is built once. Under the **element** arrangement it defaults to `off`, because React re-runs the whole component body and `{isOpen and <Panel/> or nil}` is ordinary React. The lint knows which world it is in; the lesson "Two arrangements" explains why there are two.

**Key points**

- A hole takes any expression; what it evaluates to is handed to your library once.
- `and`/`or` in a hole builds a child one time — `static_conditional_child` warns.
- `show(condition, component)` swaps a child; `indexes(source, fn)` keeps one row per index.
- The lint defaults to `warn` under the table backend and `off` under the element backend.
