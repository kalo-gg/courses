# Generics

*One type parameter turns a function that works for strings into one that works for anything.*

Luau Intermediate · Generics & Functions · 10 min

---

Every codebase has a `first`. Written for strings it is four lines; written again for numbers it is the same four lines; written with `any` it is four lines that check nothing.

A **type parameter** is the fourth option. `first<T>` introduces a name for a type the caller decides, and using that name in both the parameter and the return is what ties them together: whatever kind of list goes in, that kind of value comes out.

**`first<T>(list: { T }): T?`**

```luau
--!strict
local function first<T>(list: { T }): T?
	return list[1]
end

print(first({ 10, 20 }))
print(first({ "lamp", "rope" }))
print(first({}))
```

Prints `10`, then `lamp`, then `nil`. Three calls, three different `T`s, one implementation. The `?` is honest: an empty list has no first element, and the checker will make callers deal with that.

`T` is inferred at the call site from the argument, so nobody writing `first(names)` has to think about it. The constraint the checker enforces is that the same `T` appears in both places: a function declared `<T>(list: { T }): T?` cannot return something unrelated to what it was given.

That is the whole difference from `any`. `any` says "I do not know"; `T` says "I do not know **and neither does the caller need to**, but the two ends match".

## Saying it explicitly

**`id<<number>>(42)`**

```luau
--!strict
local function id<T>(x: T): T
	return x
end

print(id(42))              -- T inferred: number
print(id<<number>>(42))    -- T said out loud
print(id<<string>>("lamp"))
```

Prints `42  42  lamp` across three lines. The double angle brackets are Luau's explicit instantiation syntax, and they are unusual to look at because a single `<` would be ambiguous with the less-than operator.

You will rarely need it. Inference decides `T` correctly nearly always, and when it cannot, the honest fix is usually to annotate the argument instead. Explicit instantiation earns its keep for the cases where there is nothing to infer from — an empty table, a function used as a value before it is called.

One asymmetry worth remembering, because it bites people who learned it from the alias side: **a type alias may give a parameter a default, and a function may not**. `type Result<T, E = string>` is legal; `function f<T, E = string>()` is not.

**Key points**

- `<T>` after a function's name introduces a type the caller decides; using it twice ties the parameter to the return.
- `T` is inferred at the call site by matching the parameter type against the argument's.
- A generic is not `any`: `any` erases checking, `T` preserves the relationship between the two ends.
- `id<<number>>(42)` states `T` explicitly, for the rare case with nothing to infer from.
- Type **aliases** may give a parameter a default; **functions** may not.
