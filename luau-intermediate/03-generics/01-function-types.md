# Typing a function

*The arrow type, functions as parameters, and the callback signature that documents itself.*

Luau Intermediate · Generics & Functions · 9 min

---

Declaring a function uses colons. Describing the **type** of a function uses an arrow: `(number, number) -> number` is the type of anything that takes two numbers and gives back one.

You need the arrow form the moment a function becomes a value — stored in a table, passed as an argument, returned from another function. Which, in practice, is the moment a codebase stops being one file.

**`(number, number) -> number`**

```luau
--!strict
type Combine = (number, number) -> number

local function apply(a: number, b: number, f: Combine): number
	return f(a, b)
end

print(apply(3, 4, function(x, y) return x + y end))
print(apply(3, 4, function(x, y) return x * y end))
```

Prints `7`, then `12`. `x` and `y` need no annotations in the callbacks: the parameter type says what will be passed, so the checker infers them from the position the function is used in.

That last point is worth stopping on, because it is where an arrow type pays for itself. A callback whose parameter is typed is a callback whose own parameters are inferred, so the caller writes `function(x, y)` and still gets full checking inside the body.

An untyped callback parameter gives you the opposite: every lambda anyone passes has to annotate itself, and none of them are checked against each other.

**A function that takes a function**

```luau
--!strict
local function each(list: { string }, visit: (index: number, value: string) -> ())
	for i, v in list do
		visit(i, v)
	end
end

each({ "lamp", "rope" }, function(i, v)
	print(i, v)
end)
```

Prints `1  lamp`, then `2  rope`. Parameter names inside an arrow type — `index`, `value` — are optional and change nothing about the checking; they are documentation for whoever writes the callback.

## Multiple returns in a type

Luau functions return lists, so function types describe lists. One return value may be written bare — `(string) -> boolean` — and two or more are parenthesised, exactly as in a declaration.

The empty list `()` is a real return type and means "nothing useful". Write it rather than leaving the arrow's right-hand side off; a reader should not have to guess whether you forgot.

**`-> (boolean, string?)`**

```luau
--!strict
type Check = (string) -> (boolean, string?)

local function nonEmpty(s: string): (boolean, string?)
	if #s == 0 then
		return false, "empty"
	end
	return true, nil
end

local check: Check = nonEmpty
print(check("lamp"))
print(check(""))
```

Prints `true  nil`, then `false  empty`. This ok-and-message shape is the one the Structuring a Codebase module builds its error handling on, and having a name for its type is half of why it works.

**Key points**

- `(A, B) -> C` is the type of a function; the colon form declares one, the arrow form describes one.
- Typing a callback parameter means the lambdas passed to it get their own parameters inferred.
- Parameters are always parenthesised; returns are parenthesised when there is more than one.
- `-> ()` means the function returns nothing, and is worth writing rather than omitting.
- Parameter names inside an arrow type are documentation only.
