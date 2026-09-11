# Type packs and variadics

*Typing a function that takes any number of arguments, and passing that whole list along.*

Luau Intermediate · Generics & Functions · 10 min

---

A variadic function takes a list of arguments that has no fixed length. Its type cannot be a type, because there is not one of them — it is a **type pack**, a list of types, and it goes in the variadic position and nowhere else.

The simple case is a pack where every element is the same. `...: number` says "any number of numbers", and that is the annotation most variadics need.

**`...number` in a signature**

```luau
--!strict
local function sum(...: number): number
	local total = 0
	for _, n in { ... } do
		total += n
	end
	return total
end

print(sum(1, 2, 3), sum())
```

Prints `6  0`. `{ ... }` packs the arguments into a table so the loop can walk them; the annotation is on the `...` itself, not on the table.

## Generic packs

**`<A..., R...>` forwarding every argument**

```luau
--!strict
local function logged<A..., R...>(f: (A...) -> R...): (A...) -> R...
	return function(...: A...): R...
		print("calling with", select("#", ...), "arguments")
		return f(...)
	end
end

local add = logged(function(a: number, b: number): number
	return a + b
end)

print(add(2, 3))
```

Prints `calling with  2  arguments`, then `5`. `A...` and `R...` are generic packs: the wrapper takes whatever `f` takes and returns whatever `f` returns, with no idea what those are.

A pack is a list of types, not a type. That single sentence explains every rule about them: `A...` cannot annotate a local, cannot be a table's field type, and cannot appear in the middle of a parameter list. It goes last, where the variadic goes.

The two spellings are worth keeping straight. `...: number` is a pack of a known element type. `A...` is a pack the caller decides, and it is what you need whenever you are wrapping a function rather than consuming values.

> **`select("#", ...)` counts, `#{ ... }` does not**
>
> `#{ ... }` builds a table and measures it, so a `nil` anywhere in the arguments shortens the answer: for `f(1, 2, nil)` it gives **2**. `select("#", ...)` asks the call itself how many arguments it received and gives **3**. Whenever a `nil` is a legitimate argument — an optional parameter, a value forwarded from another call — `select` is the one that tells the truth.

Most variadics you write are the first kind: a logger, a formatter, a `sum`. Annotate the pack with the element type and you are done, and the checker will stop a caller passing a table where a number was meant.

The generic pack is for the rarer job of wrapping — a timer, a memoiser, a retry helper. It is worth recognising when you read one, and worth reaching for only when you genuinely do not know the signature you are wrapping.

**Key points**

- A **type pack** is a list of types, not a type; it goes in the variadic position and nowhere else.
- `...: number` types a variadic whose elements are all the same.
- `A...` and `R...` are generic packs, for wrapping a function whose signature you do not know.
- Fixed parameters come first; nothing may follow the variadic.
- `select("#", ...)` counts the arguments a call received; `#{ ... }` stops at the first `nil`.
