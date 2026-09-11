# What happens before it runs

*Source becomes bytecode, bytecode runs on a register machine, and that split explains most of the rules.*

Luau Advanced · How Luau Runs · 9 min

---

You have written Luau for two courses without needing to know what happens to it. For the next six modules you do, because every performance claim you will meet is really a claim about one of two phases.

A Luau file is **compiled** in full before a single line of it runs. The compiler folds constant arithmetic, inlines small functions where it judges that worthwhile, and decides which slot every local lives in. Then the **VM** executes the resulting bytecode, and by that point the shape of your code is fixed.

**A register and a lookup**

```luau
score = 0 -- no `local`: a global, looked up in the environment table on every read

local function countTo(n: number): number
	local total = 0 -- a local: a register the VM reads directly
	for i = 1, n do
		total += i
		score += i -- the same arithmetic, through the environment, every iteration
	end
	return total
end

print(countTo(10), score)
```

Both counters end at `55`, and they are not the same work. `total` is a register; `score` is a name that has to be found in the environment table every time it is read or written, because only a read can say what it holds right now.

That is the compile step at work: a **local** becomes a numbered slot, and a **global you assign in this script** stays a name. You would expect the standard library to behave like the second kind — `math` is a global, `sqrt` is a field of it — and in Lua 5.1 it did, which is where a well-known habit comes from.

**The exception: a library chain**

```luau
local sqrt = math.sqrt -- the Lua 5.1 habit: cache the library function in a local

local function hypotCached(a: number, b: number): number
	return sqrt(a * a + b * b)
end

local function hypotDirect(a: number, b: number): number
	return math.sqrt(a * a + b * b) -- in Luau, this costs the same
end

print(hypotCached(3, 4), hypotDirect(3, 4))
```

Both print `5`, and in Luau both do the same work. Luau resolves `math.sqrt` once, when the script is loaded — not on every call — so there is nothing left for the local to save.

Three moments matter, then, not two. At **compile** time, locals become registers and `24 * 60` becomes `1440`. When the script is **loaded**, library chains like `math.sqrt` are resolved to the functions they name — Luau calls these *imports* — on the assumption that nothing will replace `math` afterwards. At **run** time, a global you assign in this script is looked up on every read, because that is the only moment its value is known.

The load-time assumption is also why `getfenv` and `setfenv` are discouraged: calling either marks the environment as impure, and the optimization switches off for everything running in it — not just the function that made the call.

So `local sqrt = math.sqrt` above a hot loop is a Lua idiom that Luau made unnecessary. It is not wrong, and it is not faster.

## Registers, not a stack

Luau's VM is **register-based**: each function gets a block of registers, and instructions name them by number rather than pushing and popping values. That is faster per instruction and it is also where the language's hard limits come from: **200 locals** and **200 upvalues** per function, and **255 registers**.

Those are not warnings. Exceed one and the compiler rejects the file. A function that hits 200 locals is nearly always one function doing the work of three.

**What the compiler settles, and what it does not**

```luau
local SIZE = 8 * 8 -- folded: the bytecode holds 64, the multiply is gone
local count = 0

local function tick(n: number): number
	return n + SIZE -- SIZE is a local the compiler knows is never written
end

count = tick(count)
print(SIZE, count)
```

Prints `64	64`. The multiply happened once, while compiling; the addition happens every call, because `n` is only known at run time.

**Key points**

- A Luau file is compiled to bytecode in full before any of it runs.
- The compiler folds constants and assigns register slots; the VM does everything that depends on a value.
- A local read is a register; a global you assign in this script is a hash lookup on every read. A library chain like `math.sqrt` is the exception: resolved once when the script is loaded.
- The VM is register-based, which is where 200 locals, 200 upvalues and 255 registers come from.
