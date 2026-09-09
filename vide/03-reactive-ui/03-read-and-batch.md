# read and batch

*Accept a value or a source with read; group several writes into one rerun with batch.*

Vide · Reactive UI · 9 min

---

A well-behaved component should not care whether a prop is a fixed value or a source. `vide.read(v)` returns `v()` if `v` is a function and `v` otherwise, so `Text = function() return read(props.Text) end` works for both `Text = "Save"` and `Text = label` where `label` is a source.

```luau
local read = vide.read

local function Button(props: { Text: string | () -> string })
	return create "TextButton" {
		Text = function()
			return read(props.Text)
		end,
	}
end

Button { Text = "Save" }   -- fixed
Button { Text = title }    -- a source; updates live
```

`read` belongs at the boundary of a component, where a prop might be either shape. Inside the component keep reading sources directly: `read(count)` on a source you created yourself only hides that it is one. Callers get the choice; the component's own code stays honest about what is live.

## Batching writes

Every source write reruns its dependents immediately. If an effect reads both `x` and `y` and you write them one after the other, it runs twice, and the first run sees a half-updated state. `batch(fn)` holds all updates triggered inside `fn` and flushes them once when `fn` returns, so dependents run a single time with the final values.

```luau
local batch = vide.batch

local x = source(0)
local y = source(0)

root(function()
	effect(function()
		print(`x={x()} y={y()}`)
	end)
end)
-- "x=0 y=0"

x(1)
y(1)
-- "x=1 y=0" then "x=1 y=1": two runs, one with mixed state

batch(function()
	x(2)
	y(2)
end)
-- "x=2 y=2": one run
```

Batches nest safely: an inner `batch` inside an outer one flushes when the outer one ends.

**Key points**

- `read(v)` unwraps a source or returns a plain value, so props can be either.
- `batch(fn)` defers every rerun triggered inside `fn` until it returns.
- Use `batch` when several sources change together and dependents read more than one of them.
