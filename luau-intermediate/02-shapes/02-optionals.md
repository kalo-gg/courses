# Optionals and nil

*A question mark means the value may be nil, and the checker will not let you use it until you have checked.*

Luau Intermediate · Typing Data · 9 min

---

`string?` is shorthand for `string | nil`: a value that is either a string or missing. It is the type of a field that may not be set, a lookup that may not find anything, a parameter a caller may leave out.

What makes it useful is what the checker does next. It refuses to let you concatenate, index or call an optional until the code has ruled `nil` out. That refusal is the feature; every `nil` crash you have ever debugged was a missing version of it.

**`string?` and what it forces**

```luau
--!strict
type Item = { name: string, tag: string? }

local function shout(item: Item): string
	local tag = item.tag
	if tag then
		return string.upper(tag) -- here tag is a string, not a string?
	end
	return "NO TAG"
end

print(shout({ name = "rope", tag = "climbing" }))
print(shout({ name = "lamp" }))
```

Prints `CLIMBING`, then `NO TAG`. Inside the `if`, the checker has narrowed `tag` from `string?` to `string` — that is why `string.upper` is allowed there and would be reported one line above.

This is **narrowing**, and it is the whole point of the optional type. The checker follows your control flow: a branch guarded by `if tag then` knows the value is present, and the branch after it knows the value may not be.

You do not annotate anything to get this. Write the check you would have written anyway and the type gets narrower on its own.

## The two ways to satisfy it

**Check, or default**

```luau
--!strict
local function describe(tag: string?): string
	local checked = if tag ~= nil then tag else "untagged"
	local defaulted = tag or "untagged"
	return checked .. "/" .. defaulted
end

print(describe("rope"))
print(describe(nil))
```

Prints `rope/rope`, then `untagged/untagged`. Both forms produce a plain `string`; the `if` expression is explicit about the test, and `or` is shorter.

> **`or` defaults and `false`**
>
> `x or default` fires on `nil` **and** on `false`, because those are Luau's only falsy values. For a `string?` or a `number?` that never matters. For a `boolean?` it is a real bug: `local on = enabled or true` turns an explicit `false` into `true` and collapses three cases into two. Optional booleans want `if enabled == nil then` and nothing else.

There is a third option and it is usually the right one: **do not make it optional**. A `?` is a promise to every reader that the value can be missing, and every one of them now has to handle it.

If a field is set by every constructor you have, drop the `?` and let the type say so. Optionals earn their place at the edges — user input, a lookup that can fail, a parameter with a genuine default.

**Key points**

- `T?` is `T | nil`: the value may be absent, and the checker will not let you use it until you have ruled that out.
- **Narrowing** is automatic — an `if` branch that tests for `nil` sees the plain type inside it, and only inside it.
- Satisfy an optional by checking it or by defaulting it; both give you a plain `T`.
- `or` defaults fire on `false` too, so an optional boolean needs an explicit `== nil` test.
- The best fix for an optional is often to not have one: `?` at the edges, plain types inside.
