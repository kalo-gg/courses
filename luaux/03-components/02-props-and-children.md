# Props and children

*What a component receives, how it defaults a value, and how a wrapper forwards the caller's children.*

LuauX · Components · 10 min

---

The table a component receives is exactly the one the caller wrote: every attribute as a string key, every child under a numeric key. A type on the parameter says what the caller may pass and costs nothing — the compiler forwards the table untouched.

One of those string keys deserves a note now, because the project at the end of this module needs it: under the table backend an **event is an ordinary property key**, so a handler is an attribute like any other — `Activated={function() … end}` on a `TextButton`. The Backends module explains why React and Fusion spell the same thing differently. Two kinds of default follow from the same rule. A single value defaults with `or` (`props.Size or 14`); a whole table defaults with spread order, the rule from the Syntax module: what you write *before* `{props}` the caller can override, what you write *after* it you enforce.

**Forwarding children**

```luaux
local function Card(props: { Title: string })
	return (
		<Frame>
			<TextLabel TextSize={18}>{props.Title}</TextLabel>

			<!-- the caller's children, forwarded -->
			{props}
		</Frame>
	)
end
```

In **child** position, `{props}` means something related but distinct: it hands the caller's children on. Under Vide and Fluid, children arrive in the props table under numeric keys — exactly as they do for a built-in class — so passing the whole table through is how a wrapper component forwards them.

## One table, two positions

> **A spread already carries the children**
>
> Worth knowing before you write both. Under this arrangement children *are* the numeric keys of the props table, and the merge helper **concatenates** numeric keys while string keys are last-wins. So `<Frame {props} />` forwards the caller's children on its own — adding `{props}` as a child too hands the whole table over a second time.

> **This one line depends on your library**
>
> LuauX generates the code that *passes* children to a component, never the code that *reads* them back out — that part is ordinary Luau you write. So `{props}` is Vide's and Fluid's spelling; React writes `{props.children}` and Fusion `{props[Children]}`. The lesson "What ports, and what does not" covers why this is the one place a target shows through.

**Typed props, a default and forwarded children**

```luaux
export type SectionProps = {
	Title: string,
	Padding: number?,
}

local function Section(props: SectionProps)
	return (
		<Frame AutomaticSize={Enum.AutomaticSize.Y}>
			<UIListLayout Padding={UDim.new(0, props.Padding or 8)} />
			<TextLabel TextSize={18}>{props.Title}</TextLabel>
			<TextButton Text="Save" Activated={function() print(props.Title) end} />
			{props}
		</Frame>
	)
end
```

`Padding` defaults with `or`; the children come through `{props}`; and `Activated` is a property key like `Text`, holding a function instead of a string. Nothing in the markup says which is which — the type does.

**Key points**

- The props table is the caller's attributes plus children under numeric keys.
- `or` defaults a value; spread order defaults or enforces a table.
- Under the table backend an event is an ordinary property key: `Activated={fn}`.
- `{props}` in child position forwards children under Vide and Fluid; React writes `{props.children}`, Fusion `{props[Children]}`.
