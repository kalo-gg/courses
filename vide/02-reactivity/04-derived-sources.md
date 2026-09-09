# Derived sources

*derive caches a computed value so many readers share one computation and one rerun.*

Vide · Sources, Effects & Scopes · 10 min

---

A plain function that reads sources recomputes every time anyone calls it. If five effects display the same formatted string, the formatting runs five times per update. `derive(fn)` fixes that: it runs `fn` in a reactive scope, caches the result, and hands readers the cached value until a dependency changes.

## One computation, many readers

**Plain function: recomputed per reader**

```luau
local count = source(0)

local text = function()
	print("ran")
	return "count: " .. tostring(count())
end

root(function()
	effect(function() text() end)
	effect(function() text() end)
end)

count(1) -- prints "ran" twice
```

**derive: computed once, shared**

```luau
local derive = vide.derive

local count = source(0)

root(function()
	local text = derive(function()
		print("ran")
		return "count: " .. tostring(count())
	end)

	effect(function() text() end)
	effect(function() text() end)
end)

count(1) -- prints "ran" once
```

`derive` must be called inside a stable scope, like `effect`, because it creates a reactive scope.

## Unchanged results stop the cascade

A derived source also compares its new result with the old one. If the value is unchanged, dependents do not rerun. So `derive(function() return hp() > 0 end)` shields its readers from every `hp` write that keeps the answer `true`. The comparison is by identity for tables, so a derive that returns a table it mutated in place looks unchanged: return a fresh table (`table.clone`) when the contents differ.

> **Sources and derives disagree about tables**
>
> A **source** treats a written table as changed even when it is the same table (you probably mutated it). A **derive** compares its result by identity, so returning the same table it returned last time looks unchanged and nothing downstream reruns. When a derive builds a list from other sources, return a fresh table (`table.clone` or a new one) whenever the contents differ.

**Graph for the derive example**

```text
count ──▶ [root: text (derive) ──▶ effect 1
                               └──▶ effect 2 ]
```

One computation feeds two readers. Destroying the root removes `text` and both effects; `count` remains.

Reading a derived source inside an effect subscribes the effect to the *derive*, not to the sources behind it, so a derive that returns the same value as last time stops the update right there. Destroying the root removes the derive along with its readers; a derived source is a scope, which is why it needs a stable scope to be created in.

**Key points**

- `derive(fn)` returns a cached, read-only source that recomputes only when a dependency changes.
- Readers share one computation, and unchanged results do not rerun dependents.
- Use `derive` for values computed from state; never write a source from inside an effect to do the same job.
