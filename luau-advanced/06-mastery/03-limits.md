# The limits the VM really has

*200 locals, 200 upvalues, 255 registers, a bounded stack, and no tail calls at all.*

Luau Advanced · Shipping Fast Luau · 10 min

---

Module 1 named the register machine's budget in passing. Here it matters, because one of these limits is genuinely surprising and changes how recursive code has to be written.

| Limit | Value | What it means in practice |
| --- | --- | --- |
| Locals per function | 200 | A function this big is three functions; split it |
| Upvalues per function | 200 | A closure capturing this much wants a table, or a different design |
| Registers per function | 255 | Deeply nested expressions can hit this before the local count does |
| Call depth | Bounded by the stack | Deep recursion raises `stack overflow`, catchable with `pcall` |
| Tail calls | **None** | A tail-recursive function still grows the stack |

The first three are compile errors. The last two are run-time failures, which is why they are the ones that reach production.

That last row is the one to internalise. In Lua, `return f(x)` in tail position reuses the caller's frame, so a tail-recursive loop runs in constant stack space and standard advice is to write deep recursion that way.

**Luau has no tail calls.** The optimization was deliberately not implemented, so `return down(n - 1)` costs a frame exactly like any other call. Recursion in Luau is therefore always bounded by the stack, whatever shape you write it in.

**Recursion that overflows**

```luau
local function down(n: number): number
	if n == 0 then
		return 0
	end
	return down(n - 1) -- a tail call in any other Lua. Not here.
end

local ok, err = pcall(down, 100000)
print(ok, string.match(tostring(err), "stack overflow"))
```

Prints `false	stack overflow`. `pcall` catches it, which is the only reason this snippet has an output to show.

## Rewriting recursion

The transformation is mechanical. Whatever the recursive version passed down becomes an entry in a stack table; the function becomes a `while` loop that pops one entry and pushes any children. Depth then costs table memory, which is plentiful, instead of call frames, which are not.

It also changes the traversal order unless you are careful — popping from the end of an array gives you depth-first in reverse. For a sum that does not matter; for anything order-sensitive, it does.

**The same traversal, iterative**

```luau
type Node = { value: number, children: { Node } }

local function sumTree(root: Node): number
	local stack: { Node } = { root }
	local total = 0
	while #stack > 0 do
		local node = table.remove(stack) :: Node
		total += node.value
		for _, child in node.children do
			table.insert(stack, child)
		end
	end
	return total
end

local tree: Node = {
	value = 1,
	children = { { value = 2, children = {} }, { value = 3, children = {} } },
}
print(sumTree(tree))
```

Prints `6`. There is exactly one call frame in play however deep the tree goes.

**Key points**

- 200 locals, 200 upvalues and 255 registers per function are compile errors when exceeded.
- Luau has no tail calls, so a tail-recursive function still grows the stack.
- Deep recursion therefore has to become a loop with an explicit stack, or a walk over a link.
- `pcall` catches a stack overflow, which makes it observable but does not make the traversal finish.
