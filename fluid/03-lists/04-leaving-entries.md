# Entries that leave slowly

*An entry is told when it has been removed, and can ask for time before it goes.*

Fluid · Lists & Control Flow · 12 min

---

`active` is a readable that goes false the moment an entry's value leaves the input table. On its own that is only information. Paired with the transform's **second return value** — a delay in seconds — it becomes the thing Fluid can do that the others cannot.

**Asking for time before you go**

```luau
local destroy = fluid.root(function()
	local items = fluid.source({ "Sword", "Shield" })

	local entries = fluid.for_values(items, function(value, key, active)
		local element = fluid.create("TextLabel")({
			Text = value,
			-- fade out when active goes false
			BackgroundTransparency = function()
				return if active() then 0 else 1
			end,
		})
		return element, 0.5 -- half a second before I am destroyed
	end)

	print(#entries()) --> 2

	items({ "Sword" })
	print(#entries()) --> still 2, until the half second is up

	step(60)
	print(#entries()) --> 1
end)

destroy()
```

Return the output and a delay. Fluid keeps the entry alive for that long after it leaves, with `active` reading false the whole time. The list is lazy, so read it once before the removal or the departing entry is never built.

| Moment | In the output? | `active()` |
| --- | --- | --- |
| While the value is in the input | Yes | `true` |
| Just after it is removed | **Yes** | `false` |
| Once the delay has elapsed | No | — destroyed |

Without a delay, the middle row does not exist: the entry leaves at once.

## The list only catches up on a step

A reactive child list is a **deferred** node, not an eager one. Writing a new table into the source marks the list stale; the instances under the Frame are not touched until the scheduler flushes deferred nodes — the middle phase of one `step`. So `active`-driven properties do not change on the same line as the write. They change one frame later.

**One frame of lag, and it is deliberate**

```luau
local items = fluid.source({ "a", "b", "c" })

local destroy, panel = fluid.root(function()
	return fluid.create("Frame")({
		fluid.for_values(items, function(value, key, active)
			return fluid.create("TextLabel")({
				Name = value,
				TextTransparency = function()
					return if active() then 0 else 1
				end,
			}), 1
		end),
	})
end)

items({ "a" })
print(#panel:GetChildren(), panel.b.TextTransparency) --> 3  0
step(1)
print(#panel:GetChildren(), panel.b.TextTransparency) --> 3  1
step(70)
print(#panel:GetChildren())                           --> 1
```

Three rows the whole time; the transparency flips on the first step and the rows are destroyed after the delay. Property bindings are eager, but the *list* is deferred, and the list is what tells `active` to change.

> **Read the list once before the removal**
>
> The list node is lazy: if nothing has ever read it, the departing entry was never built, and there is nothing to keep alive. In a real screen the `create` table reads it for you; in a console example, read it first.

One more consequence of keying by value: an entry survives a rewrite only if its value is the **same value**. Write a fresh table for a row that did not change and Fluid sees the old one leaving and a new one arriving — two rows, a fade you did not ask for, and a rebuilt scope. Keep the identities stable.

**Key points**

- The transform's second return value is a delay in seconds; without one, a removed entry leaves at once.
- `active` goes `false` the moment the value leaves the input, and stays false for the whole delay.
- Bind a transparency, size or spring to `active` and the row animates itself out.
- Reactive child lists are **deferred**: the tree catches up on the next scheduler step, not on the line that wrote the source.
- The list is lazy — read it at least once, or the entry you expected to linger was never built.
