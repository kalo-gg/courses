# Fragments, spreads and comments

*Several siblings without a wrapper, a props table forwarded in one hole, and notes that never reach the output.*

LuauX · The Syntax · 9 min

---

A fragment returns several siblings with no wrapper element, for when a component's children belong to whatever contains it rather than to a `Frame` of their own.

**Fragment**

```luaux
<>
	<TextLabel>One</TextLabel>
	<TextLabel>Two</TextLabel>
</>
```

## Spreads

A spread forwards a whole props table into an element. The rule that makes it useful is **source order — later wins**. Write your value *after* the spread and it beats the caller's; write it *before* and the caller's wins. That one decision is the difference between a value a component enforces and a default it merely suggests.

**A default the caller can override**

```luaux
local function PrimaryButton(props: ButtonProps)
	return (<Button {props} Color={Color3.fromRGB(0, 120, 255)} />)
end
```

`Color` comes after the spread, so it beats any `Color` in `props`. Move it before and the caller wins instead.

## Comments

Comments have two spellings inside markup and one trap. `<!-- … -->` works between tags, and `{--[[ … ]]}` works in child position for the times a Luau comment reads more naturally. Neither reaches the output. The trap is `--`: it does **not** start a comment inside markup, where it is ordinary text, so a stray `-- note` between the tags of a label ends up on screen.

**Two comments and one label**

```luaux
<TextLabel>
	<!-- a markup comment: never emitted -->
	{--[[ a Luau comment in child position: never emitted ]]}
	-- not a comment here: this is the label's text
</TextLabel>
```

**Key points**

- A fragment returns siblings with no wrapper.
- A spread forwards a whole table; whatever is written later wins.
- Default before the spread, enforced value after it.
- `<!-- -->` and `{--[[ ]]}` are comments; `--` between tags is text.
