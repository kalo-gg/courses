# Components are functions

*One table in, an element out, indistinguishable from a built-in class at the call site.*

LuauX · Components · 10 min

---

There is no component API. A component is **any function taking a single table**, and the table it receives is the one the caller wrote as attributes and children. Nothing registers it, nothing wraps it, and it has no lifecycle of its own.

**A component**

```luaux
export type ButtonProps = {
	Label: string | vide.Source<string>,
	OnClick: (...any) -> (),
	Color: Color3?,
}

local function Button(props: ButtonProps)
	return (
		<TextButton
			Size={UDim2.fromOffset(160, 40)}
			BackgroundColor3={props.Color or Color3.fromRGB(30, 30, 30)}
			Activated={props.OnClick}
			Text={props.Label}
		>
			<UICorner CornerRadius={UDim.new(0, 8)} />
		</TextButton>
	)
end
```

Used as `<Button Label="Click me" OnClick={…} />` — the same shape as any Roblox class.

Look at `Label`. Its type admits a plain string **or** a source, and the component handles neither case: it passes the value straight through to the attribute. The library decides what to do with what arrives — a function on a property key is reactive under Vide and Fluid, and a plain value is an assignment. Writing components that accept both therefore costs nothing.

## How a tag name is resolved

A tag resolves in three steps: a **`luaux.toml` alias** first, then a **Roblox class**, then anything bound in the file — and otherwise it errors with a suggestion. The middle step is the one you trip over, because Roblox's class list is longer than anyone remembers. The first matters because a rename is exclusive: once `[elements]` declares `TextLabel = "text"`, `<TextLabel>` is an error rather than a class.

**Why this is not called `Dialog`**

```luaux
-- Roblox ships a class named Dialog — the NPC conversation instance.
-- <Dialog /> would build one of those, not call this function.

local function ConfirmDialog(props: ConfirmDialogProps)
	return (<Frame>…</Frame>)
end
```

Straight out of `examples/ConfirmDialog.luaux`, where the name is chosen for exactly this reason.

> **A dotted name is always a component**
>
> `<Foo.Bar />` skips class resolution entirely — no Roblox class has a dot in its name. So a component kept on a table (`<Styles.Panel />`, `<UI.Card />`) can never collide with anything Roblox ships, which is a reasonable habit for a library of shared components.

**Key points**

- A component is any function taking one table; nothing registers it.
- A tag resolves alias → Roblox class → file binding; a class always beats a local.
- A dotted name (`<UI.Card />`) skips class resolution entirely.
- Props pass straight through: a value or a function, the library decides.
