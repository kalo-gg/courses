# Patterns that scale

*Conventions for props, state ownership, lists and performance from real Vide projects.*

Vide · Shipping a Vide App · 9 min

---

Vide is small enough that most of "doing it right" is habit. These are the habits that keep a growing UI readable and fast.

## Props and state ownership

```luau
type Props = {
	Text: string | () -> string,
	Disabled: (boolean | () -> boolean)?,
}

local function Button(props: Props)
	return create "TextButton" {
		Text = function() return read(props.Text) end,
		AutoButtonColor = function() return not read(props.Disabled) end,
	}
end
```

Callers can pass a literal today and a source tomorrow without touching the component.

Type your props. Vide exports the types for it: a prop the component only reads is `() -> T`, one it also writes is `vide.Source<T>`, and a plain `T` is a snapshot that will never update. Writing the table out — `{ Text: () -> string, count: vide.Source<number>, Size: UDim2? }` — states the contract at the top of the file, and it is exactly the shape `read` exists to accept.

Create a source in the highest component that needs to change it, and pass it *down*. Sibling components that both need a value read the same source; they never copy it. Values needed everywhere go in a context. Game state that many systems touch belongs outside the UI entirely: expose sources from a state module and have the UI read them.

**A state module the game owns and the UI reads**

```luau
-- state.luau
local vide = require(ReplicatedStorage.Packages.vide)
local source = vide.source

return {
	health = source(100),
	coins = source(0),
}

-- Hud.luau: reads only
local state = require(script.Parent.state)

create "TextLabel" {
	Text = function()
		return "HP " .. state.health()
	end,
}
```

## Derive, never write from effects

If a value is computed from other sources, use `derive`. An effect that writes a source can trigger itself through another effect and is the number one cause of "why did this run twice". Effects should touch instances, connections or the outside world, not other sources.

## Lists and bursts

Use `values` for items with identity so rows move rather than rebuild, and `indexes` for positional slots or repeatable data; either way, update a list by writing the source. `batch` bursts of writes, and `derive` an expensive calculation once if several bindings need it.

> **Coming from Fusion or Roact**
>
> Fusion's `Value`, `Computed` and `Observer` are Vide's `source`, `derive` and `effect`, and its `[Children]` key is Vide's unkeyed slot. Roact's `setState` has no equivalent, because Vide never re-renders a component: only the bindings that read a changed source rerun. So port the state first — every `Value` or piece of `state` becomes a `source` in one module — keep components as plain functions returning `create`, and delete everything that existed to make re-rendering cheap: keys, `shouldUpdate`, memoised children. `indexes` and `values` do that job now.

**Key points**

- Accept values or sources in props and unwrap with `read`.
- Own state high, pass it down, share sources rather than copying values.
- Compute with `derive`; reserve effects for the outside world.
- Pick `values` or `indexes` by identity, and batch bursts of writes.
- Type props with `() -> T` when the component only reads and `vide.Source<T>` when it writes; a bare `T` is a snapshot.
