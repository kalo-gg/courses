# Sources and scopes

*State is a function you call, and almost everything else needs a scope to live in.*

Fluid · Fluid Foundations · 11 min

---

A **source** is a function that does double duty. Call it with no arguments and it returns its value; call it with one and it sets it. That is the whole interface — there is no `:get` or `:set`.

**Reading and writing**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local coins = fluid.source(10)

print(coins()) --> 10
coins(25)
print(coins()) --> 25
```

## Why most things need a root

Sources work anywhere. Almost nothing else does: the rest of the library needs a **stable scope** — somewhere to belong, so Fluid knows when to tear it down. `fluid.root` creates one. The demonstration below uses `derive`, which computes a value from sources; it is the first lesson of the next module, and here it is only standing in for the whole list.

**The error you will meet first**

```luau
local coins = fluid.source(25)

-- outside a root: fails
local ok, err = pcall(function()
	return fluid.derive(function() return coins() * 2 end)
end)
print(err) --> cannot call derive without a parent scope

-- inside one: fine
local destroy = fluid.root(function()
	local doubled = fluid.derive(function() return coins() * 2 end)
	print(doubled()) --> 50
end)

destroy()
```

The message names the function you called, and no more. `root` returns the destructor first, then whatever the function returned.

The rule is wider than `derive` and `effect`. Everything in this list calls the same check, and every one of them fails at the top level: `derive`, `effect`, `deferred`, `async`, `interval`, `for_values`, `for_keys`, `switch_delay` (and `show_delay` through it), `tags`, `lerp` and `spring` — plus `create` itself, whenever **any** property in its table is a function.

> **There is a second version of that error**
>
> The one above is for the top level. The other appears when the surrounding scope exists but is *reactive* — you built a component, a `derive` or a reactive `create` from inside an `effect`, a `derive` or a property function. It reads **`cannot call <name> while the parent scope is reactive - this might be a component. consider using untrack() while creating the components.`** The fix is in the message.

> **And one message that is still a placeholder**
>
> A signal-valued property (`Activated = fn`) and `fluid.changed` only need *a* scope, not a stable one, so they go through a different assertion — and outside a root they fail with the literal string **`reactive scope error thing, TODO error message`**.

**Key points**

- A source is one function: no argument reads (and subscribes), any argument writes.
- `fluid.root(fn)` creates a stable scope and returns `(destroy, fn's returns)` — destructor first.
- `derive`, `effect`, `deferred`, `async`, `interval`, `for_values`, `for_keys`, `show_delay`, `switch_delay`, `tags`, `lerp`, `spring` and any `create` with a function property all need one.
- Outside a root: `cannot call <name> without a parent scope`. Inside a *reactive* one: `… while the parent scope is reactive`, and `untrack` is the fix.
- Signal properties and `changed` fail with the unfinished `reactive scope error thing, TODO error message`. It means the same thing.
