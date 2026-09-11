# Luau Advanced reference

*Luau Advanced 0.733, released August 7, 2026. Checked on September 9, 2026.*

---

## Compilation and execution

What happens between your source file and the register machine that runs it.

### `The compile step`

```luau
source -> bytecode -> VM
```

Luau compiles a whole source file to bytecode before anything runs. Constant arithmetic is folded, small functions may be inlined, and the slot of every local is decided. Nothing in that phase looks at run-time values.

> A global you assign in this script survives compilation as a table lookup in the environment; a local becomes a register number. A library chain like `math.sqrt` is the exception: Luau resolves it once when the script is loaded, so `local sqrt = math.sqrt` buys nothing.

### `--!native`

```luau
--!native   -- first lines of a file
```

Asks the host to compile every function in the file to machine code. It parses and runs in the sandbox; whether native code is actually produced is the host's decision, and neither Luau nor this course can report it from inside the script.

> It is a hint, never a guarantee. Nothing in the language observes whether codegen ran.

### `@native`

```luau
@native
local function f(...) end
```

The same request for one function, written as an attribute on its declaration. Use it when a single function in an otherwise cold file is the hot one.

> Attributes attach to the `function` declaration, so `@native` goes on its own line directly above `local function` or `function`.

### `--!optimize`

```luau
--!optimize 0 | 1 | 2
```

Sets the optimization level for the file. `2` turns on the whole set, including inlining and loop unrolling where the compiler judges them worth it. It parses and runs in the sandbox.

> Level 2 is what shipped Roblox code runs; level 0 exists for debugging odd codegen, not for production.

### `VM limits`

```luau
200 locals · 200 upvalues · 255 registers per function
```

Luau is a register machine, and those registers are the budget of one function. Exceed any of the three and the compiler rejects the file rather than degrading quietly.

> Hitting these usually means one function is doing the work of three. Split it before reaching for a table of values to dodge the local limit.

## Advanced types

The annotations you write when the easy answer does not fit. All of these parse and run in the sandbox; only an analyzer reports on them.

### `Refinement`

```luau
if typeof(x) == "string" then ... end
```

The checker follows control flow: inside the branch, `x` is narrowed to the type the guard proved. `type(x)`, `x == nil`, `x ~= nil` and plain truthiness tests all narrow.

> `typeof` is the Roblox-flavoured `type`. In plain Luau it agrees with `type` on every value except that it names `vector` and `buffer`.

### `assert narrowing`

```luau
assert(v ~= nil, "message")
```

Narrows for the rest of the scope rather than for one branch, because the code after it only runs when the condition held.

> Assign the field to a local first and assert on the local: a refinement on `t.field` is thrown away by the next function call, and a local's is not.

### `Refinement invalidation`

```luau
if t.x ~= nil then f() print(t.x) end
```

A call between the check and the use can invalidate the refinement, because the checker cannot prove the call did not write to what was narrowed. Reassignment invalidates it too.

> The same applies to an upvalue any function in scope could write.

### `Intersection`

```luau
type Row = Named & Counted
```

A value that is both shapes at once — the mirror of a union. A table satisfies it when it satisfies every member.

> An intersection of function types is how Luau spells an overload: `((number) -> string) & ((string) -> string)`.

### `Variance`

```luau
{ Dog } is not assignable to { Animal }
```

Mutable containers are invariant: allowing the assignment would let someone push a plain `Animal` into a list the other side reads as `Dog`s. Functions go the other way — one taking an `Animal` can stand in where one taking a `Dog` is wanted.

### `read / write modifiers`

```luau
type Frozen = { read legs: number }
```

Marks a property read-only (or write-only) in the type. A read-only container is safe to widen, so `{ read Dog }` is assignable to `{ read Animal }` where the mutable version is not.

### `type function`

```luau
type function name(t)
  return t
end
```

A function that runs at check time and returns a type. Its body sees the argument as a value of the `types` library and can inspect and build types with it.

> The newest corner of the language: the RFCs are the reference, and behaviour may still move. It parses and runs in 0.733; nothing about it is observable at run time.

### `keyof / index`

```luau
keyof<T>   index<T, K>
```

`keyof<T>` is the union of `T`'s keys; `index<T, K>` is the type stored at key `K`. Together they type a lookup without repeating the table's shape.

> `keyof` orders the union it produces lexicographically as of 0.737, which only matters when you print the type in an error message.

## buffer

A fixed-size block of raw bytes. Not a table: no keys, no iteration, no metatable — you decide the layout.

### `buffer.create`

```luau
buffer.create(size: number): buffer
```

Allocates `size` bytes, all zero. The size is fixed for the buffer's life.

### `buffer.fromstring`

```luau
buffer.fromstring(s: string): buffer
```

A buffer holding the bytes of `s`, so `buffer.len` equals `#s`.

### `buffer.tostring`

```luau
buffer.tostring(b: buffer): string
```

The buffer's bytes as a string. Round-trips with `fromstring`.

### `buffer.len`

```luau
buffer.len(b: buffer): number
```

The size in bytes. `#b` does not work on a buffer; this is the only way to ask.

### `buffer.copy`

```luau
buffer.copy(target, targetOffset, source, sourceOffset?, count?)
```

Copies `count` bytes between buffers. Overlapping ranges within one buffer are handled correctly.

### `buffer.fill`

```luau
buffer.fill(b, offset, value, count?)
```

Writes the byte `value` `count` times from `offset`, or to the end when `count` is omitted.

### `buffer.readu8 / writeu8`

```luau
buffer.readu8(b, offset): number
buffer.writeu8(b, offset, value)
```

One unsigned byte, 0–255. The cheapest field there is; a flags byte or a small enum belongs here.

> Writing a value outside the range wraps rather than erroring. Reading past the end raises `buffer access out of bounds`, which is the good news: a buffer never hands back garbage.

### `buffer.readu16 / writeu16`

```luau
buffer.readu16(b, offset): number
```

Two bytes, unsigned. Every read and write in the buffer library is little-endian regardless of the host.

### `buffer.readu32 / writeu32`

```luau
buffer.readu32(b, offset): number
```

Four bytes, unsigned. The usual choice for an id or a packed bitfield.

### `buffer.readi8 / i16 / i32`

```luau
buffer.readi32(b, offset): number
```

The signed family, in one, two and four bytes. Same offsets, different reading of the top bit.

### `buffer.readf32 / f64`

```luau
buffer.readf32(b, offset): number
buffer.writef32(b, offset, value)
```

Four-byte and eight-byte floats. `f32` halves the size of a position or velocity field and is what a simulation normally wants.

> `f32` rounds: `1.5` and `2.5` survive exactly, `0.1` does not. Use `f64` when the value must come back bit-identical.

### `buffer.readstring / writestring`

```luau
buffer.readstring(b, offset, count): string
buffer.writestring(b, offset, value, count?)
```

Raw bytes as text. There is no length prefix and no terminator: you have to know or store the count yourself.

## vector

Three numbers the VM treats as one value. Core Luau, not Roblox — `Vector3` is a different type with a different library.

### `vector.create`

```luau
vector.create(x: number, y: number, z: number): vector
```

Builds a vector. `typeof` reports `vector`, and printing one gives its three components separated by `, `.

> Components read as `v.x`, `v.y`, `v.z` and are read-only: assigning to one raises `attempt to index vector with 'x'`.

### `vector.zero / vector.one`

```luau
vector.zero   vector.one
```

Constants, not functions: `(0, 0, 0)` and `(1, 1, 1)`.

### `vector.dot`

```luau
vector.dot(a: vector, b: vector): number
```

The scalar product. Zero for perpendicular vectors, which is the cheapest way to ask whether two directions disagree.

### `vector.cross`

```luau
vector.cross(a: vector, b: vector): vector
```

A vector perpendicular to both. The cross of the x and y axes prints `0, 0, 1`.

### `vector.magnitude`

```luau
vector.magnitude(v: vector): number
```

The length. The magnitude of `(3, 4, 0)` is `5`.

### `vector.normalize`

```luau
vector.normalize(v: vector): vector
```

The same direction, length one. Multiply the result to get a vector of a chosen length.

> Components are single-precision, so normalizing `(0, 3, 4)` prints `0, 0.6000000238418579, 0.800000011920929` even though its magnitude comes back as exactly `1`.

### `vector.angle`

```luau
vector.angle(a: vector, b: vector, axis: vector?): number
```

The angle between two vectors in radians; with an axis, a signed angle around it.

### `vector.lerp`

```luau
vector.lerp(a: vector, b: vector, t: number): vector
```

Linear interpolation between two vectors.

### `vector.clamp`

```luau
vector.clamp(v: vector, min: vector, max: vector): vector
```

Clamps each component into the box the two bounds describe. `vector.min` and `vector.max` do the same job one side at a time.

### `Vector arithmetic`

```luau
a + b   a - b   -a   v * n   v / n
```

The operators work component-wise, with a number scaling every component. `pos + vel * dt` is one expression.

> A vector is a value, not an object: passing one around and doing arithmetic on it never touches the heap, which is why a stepped simulation over vectors can report zero heap growth.

## Immutability

Freezing is how a module hands out a table it still trusts an hour later.

### `table.freeze`

```luau
table.freeze(t: T): T
```

Makes `t` read-only, in place, and returns it. A later write raises `attempt to modify a readonly table`.

> Shallow: `table.freeze({ tags = {} })` leaves `tags` fully writable. Freeze the nested tables too if you mean it.

### `table.isfrozen`

```luau
table.isfrozen(t): boolean
```

Whether the table itself is frozen. It says nothing about its values.

### `table.clone`

```luau
table.clone(t: T): T
```

A shallow copy with the same metatable. The usual way to hand out an editable version of a frozen table.

> The result is never frozen, even when the source was, and nested tables are shared rather than copied.

### `table.create / table.clear`

```luau
table.create(count: number, value: T?): { T }
table.clear(t)
```

Sizes an array once instead of growing it, and empties one without allocating a replacement. The pair is how a hot loop reuses a table instead of making one per iteration.

## coroutine

Functions that stop in the middle and start again later. Cooperative: nothing here runs in parallel.

### `coroutine.create`

```luau
coroutine.create(f: (...) -> ...): thread
```

Wraps a function as a suspended coroutine. Nothing runs until the first `resume`.

### `coroutine.resume`

```luau
coroutine.resume(co, ...): (boolean, ...)
```

Runs the coroutine until it yields or returns. The first result says whether it survived; the rest are what it yielded or returned.

> It never throws. An error inside comes back as `false` plus the message, and a resume of a dead coroutine as `false, "cannot resume dead coroutine"`.

### `coroutine.yield`

```luau
coroutine.yield(...): ...
```

Suspends the running coroutine, handing its arguments to whoever resumed it. It returns when someone resumes again, with that resume's arguments.

> It cannot cross a metamethod or a C-call boundary: yielding inside `__index` raises `attempt to yield across metamethod/C-call boundary`.

### `coroutine.wrap`

```luau
coroutine.wrap(f): (...) -> ...
```

The same machinery behind a plain function. Calling it resumes; errors propagate to the caller instead of being returned.

> There is no thread to hold, so `status` and `close` are not available for a wrapped coroutine.

### `coroutine.status`

```luau
coroutine.status(co): "suspended" | "running" | "normal" | "dead"
```

Where the coroutine is. `normal` means it resumed another coroutine and is waiting for it.

### `coroutine.close`

```luau
coroutine.close(co): (boolean, string?)
```

Releases a suspended coroutine and marks it dead, running nothing further inside it.

> Closing an already-dead coroutine is fine and returns `true`. Resuming a closed one returns `false, "cannot resume dead coroutine"`.

### `coroutine.isyieldable`

```luau
coroutine.isyieldable(): boolean
```

Whether a `yield` from here would work. Useful inside a library that may be called from either a coroutine or the main thread.

> What counts as yieldable depends on how the host started your chunk; in the Kalo sandbox the top level of a script already reports `true`.

## Metamethods

The hooks past `__index`. Every one of these was fired in the sandbox to check what it actually does.

### `Arithmetic and concatenation`

```luau
__add __sub __mul __div __idiv __mod __pow __unm __concat
```

Called when an operator meets a value the VM cannot handle itself. Luau tries the left operand's metatable first, then the right one's, so `Vec2 + Vec2` and `Vec2 + number` can share one handler.

> `__idiv` backs `//`, and integer division by zero is `inf` in Luau rather than an error.

### `Comparison`

```luau
__eq __lt __le
```

`__lt` and `__le` back `<` and `<=` (and, by negation, `>` and `>=`). `__eq` is consulted only when both operands are tables.

> Luau calls `__eq` even when the two operands are the same object, which differs from Lua. It does not call it when the two operands carry different `__eq` handlers — that comparison falls back to raw identity.

### `__call`

```luau
__call(self, ...) -> ...
```

Makes a table callable. A module can then expose one obvious operation as `M(x)` and still carry configuration as fields.

### `__len`

```luau
__len(self) -> number
```

Backs `#`. Useful for a collection whose real count is not its array part.

> `rawlen` bypasses it, and the `table` library does not consult it: `table.insert` and `table.concat` both work on the raw array part.

### `__tostring`

```luau
__tostring(self) -> string
```

What `tostring`, `print` and string concatenation show. The cheapest debugging investment a custom type can make.

### `__iter`

```luau
__iter(self) -> (iterator, state, control)
```

What a generic `for` consults before falling back to iterating the table itself. It is how a wrapper type stays loopable without exposing its storage.

> It replaced Lua 5.2's `__pairs`, which Luau does not call at all.

## Measuring, and what this sandbox has

The honest list: what you can measure here, and what you have to leave the browser for.

### `gcinfo`

```luau
gcinfo(): number
```

Live heap in whole kilobytes. Available in the Kalo sandbox, and the only real measurement this course takes: read it either side of a loop and the difference is the memory that loop kept.

> Whole kilobytes, so a single small allocation can read as zero growth; and a collection during the window can make the difference negative, which is still no growth.

### `os.clock and the microprofiler`

```luau
os.clock(): number   -- NOT available in this sandbox
```

The real timing tools. `os.clock` is the standalone-Luau one (Lune, the `luau` CLI); Roblox Studio's microprofiler is the in-engine one, driven by `debug.profilebegin` and `debug.profileend`.

> The Kalo sandbox exposes neither `os` nor `debug`, so nothing in this course times anything. Every number a lesson prints is an operation count or a `gcinfo` reading, and the lesson says which.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Luau CLI](https://github.com/luau-lang/luau) | Run and analyze Luau outside Roblox; `--codegen` turns on native compilation | 0.737 | `rokit add luau-lang/luau` |
| [Lune](https://github.com/lune-org/lune) | Standalone Luau runtime with a standard library (fs, net, process, task) | 0.10.5 | `rokit add lune-org/lune` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Language server: refinements, intersections and type functions checked in your editor | 1.69.0 | `rokit add JohnnyMorganz/luau-lsp` |
| [StyLua](https://github.com/JohnnyMorganz/StyLua) | Formatter | 2.5.2 | `rokit add JohnnyMorganz/StyLua` |
| [Selene](https://github.com/Kampfkarren/selene) | Linter; catches the shadowed locals and unused upvalues a 200-local function accumulates | 0.31.0 | `rokit add Kampfkarren/selene` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- Lua 5.4 documents integer division and modulo by zero as errors. Luau has one number type and follows floating point: `1 // 0` is `inf` and `1 % 0` is `nan`.
- Lua 5.3 added the bitwise operators `&`, `|`, `~` and `<<`. Luau never adopted them — `&` in Luau source is a type intersection, and bit work goes through the `bit32` library.
- `goto` and labels appear in Lua 5.2 tutorials as the way out of a nested loop. Luau has never had them; `goto` is a syntax error. Use a flag, an early `return`, or a small function.
- "Use a tail call for deep recursion" is standard Lua advice and wrong here: Luau has no tail calls. An explicitly tail-recursive function still grows the stack and still overflows, so deep recursion has to become a loop or an explicit stack.
- Older Roblox material calls `Vector3` "the Luau vector type". Core Luau's `vector` is a separate value type with its own library; a Roblox `Vector3` is an engine datatype and is not what `vector.create` returns.
- Almost every performance article reaches for `os.clock` as though it were always available. It is a standard library function the host may not expose — this sandbox does not.
- `__pairs` shows up in Lua 5.2-era guides for custom iteration. Luau replaced it with `__iter`, which is what a generic `for` consults; `__pairs` is not called at all.
