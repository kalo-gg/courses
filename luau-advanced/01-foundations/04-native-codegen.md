# Native code generation

*What --!native and @native ask for, which functions actually gain from it, and which cannot.*

Luau Advanced · How Luau Runs · 10 min

---

Everything so far assumed the VM interprets bytecode. It does not have to. Luau can compile functions to **machine code** ahead of running them, and you ask for that with `--!native` at the top of a file or `@native` on one function. Both parse and run in 0.733 — including here.

**The directive, and the attribute**

```luau
--!native
-- `--!native` at the top of a file asks for every function in it.

@native
local function distance(x1: number, y1: number, x2: number, y2: number): number
	local dx, dy = x2 - x1, y2 - y1
	return math.sqrt(dx * dx + dy * dy)
end

print(distance(0, 0, 3, 4))
```

Prints `5`, with or without codegen. That is the point: the annotation changes how the function is compiled, never what it computes.

Native compilation pays off for **arithmetic-heavy, type-stable** code: numbers in, numbers out, few table touches. It does very little for code dominated by table access, string building or metamethod dispatch, because those still go through the same runtime routines whether the caller was interpreted or native.

It is also a *request*. The host decides whether code generation is on at all — Studio has a setting, the `luau` CLI has `--codegen` — and nothing inside the language can report whether it happened.

## What it asks of your code

| Shape | Effect of codegen | Why |
| --- | --- | --- |
| Numbers in, numbers out, stable types | The best case | Arithmetic becomes machine arithmetic instead of interpreted instructions |
| Heavy table indexing | Little | Lookups still call the same runtime routines |
| String building | Little | Allocation and hashing dominate, and both are runtime work |
| Varargs and metamethod dispatch | Little to none | These deoptimize back into the interpreter's paths |
| Any of the above | Never a guarantee | The host decides whether codegen runs at all |

**One function worth marking, in a file that is not**

```luau
-- No `--!native` at the top: most of this module is table and string work.

local registry = {}

local function register(name: string, value: any)
	registry[name] = value -- table work; codegen has nothing to remove here
end

@native
local function integrate(pos: number, vel: number, dt: number): number
	return pos + vel * dt -- numbers in, a number out, called per entity per frame
end

register("gravity", -9.81)
print(registry.gravity, integrate(0, 10, 0.5))
```

Prints `-9.81	5`. Marking the file would ask for codegen on `register` too, which would gain nothing; the attribute names the one function that qualifies.

A last piece of honesty: this sandbox cannot tell you whether codegen ran, so the exercise below is checked by reading your code rather than by measuring anything. That is the correct check for this feature — the decision is a judgement about the *shape* of a function, and the shape is what you can inspect.

**Key points**

- `--!native` marks a whole file, `@native` marks one function; both parse and run in Luau 0.733.
- The gain comes from arithmetic on stable number types, not from doing more work in general.
- Table access, string building, varargs and metamethod dispatch gain little or nothing.
- It is a hint: the host decides whether codegen happens, and no code can observe that it did.
