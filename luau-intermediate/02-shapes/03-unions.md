# Unions and literal types

*A value that is one of several shapes, and the string literals that let you tell them apart.*

Luau Intermediate · Typing Data · 10 min

---

A union type, written `A | B`, is a value that is one of several types. You have already used one: `string?` is `string | nil`.

The rule for using a union is the rule for using an optional. The checker only lets you do what is safe for **every** member until you have narrowed it down, so a `number | string` supports almost nothing until you test which one you have.

**`number | string`, and why you must narrow it**

```luau
--!strict
local function show(v: number | string): string
	if type(v) == "number" then
		return "n=" .. v * 2   -- v is a number in here
	end
	return "s=" .. string.upper(v) -- and a string out here
end

print(show(21))
print(show("lamp"))
```

Prints `n=42`, then `s=LAMP`. `type(v) == "number"` is what the checker follows; multiplying before that test would be reported, because a `string` cannot be multiplied.

## Literal types

A string literal is a type in its own right. `"add"` is a type with exactly one value in it, and `"add" | "remove"` is the classic command type: the checker rejects `"delete"` at the call site instead of letting a typo reach a dispatch table.

Booleans work the same way — `true` and `false` are types. **Numbers are not**: `type N = 1` is a syntax error in Luau, and there is no plan to add it. When you want a fixed set of numeric options, name them with strings and map to numbers inside.

**A tagged union**

```luau
--!strict
type Add = { kind: "add", name: string, qty: number }
type Remove = { kind: "remove", name: string }
type Command = Add | Remove

local function apply(store: { [string]: number }, cmd: Command)
	if cmd.kind == "add" then
		store[cmd.name] = (store[cmd.name] or 0) + cmd.qty
	else
		store[cmd.name] = nil
	end
end

local store: { [string]: number } = {}
apply(store, { kind = "add", name = "lamp", qty = 2 })
apply(store, { kind = "remove", name = "lamp" })
print(store.lamp)
```

Prints `nil`. The shared `kind` field is the **tag**: testing it narrows `cmd` to one member, which is why `cmd.qty` is legal in the first branch and would be an error in the second.

Two habits make tagged unions pleasant. Give every member the **same** tag field name, so one test serves them all. And put the tag first in the type, so the shape reads as "a command, of this kind, with these extras".

The payoff arrives in the Metatables and Iterators modules, where a dispatch function on a tagged union replaces a chain of `if type(x) ==` tests that nothing was checking.

**Key points**

- `A | B` is a value that is one of several types; you may only do what is safe for every member until you narrow.
- String literals and booleans are types; `"add" | "remove"` is a checked set of options.
- **Number literal types do not exist in Luau** — `type N = 1` is a syntax error.
- A **tagged union** gives every member the same `kind` field with a distinct literal, so one test splits them.
- `type(v) == "number"` and `cmd.kind == "add"` are both narrowing tests the checker follows.
