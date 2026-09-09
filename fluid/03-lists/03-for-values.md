# for_values and for_keys

*Map a table's values, and learn what the transform is handed and what the output is not.*

Fluid · Lists & Control Flow · 12 min

---

`for_values` maps the values of a table into a new table. The interesting part is the transform's signature, which carries two things Vide and Fusion do not give you.

**The transform's three arguments**

```luau
local fluid = require(game:GetService("ReplicatedStorage").Packages.fluid)

local destroy = fluid.root(function()
	local items = fluid.source({ "Sword", "Shield" })

	local labels = fluid.for_values(items, function(value, key, active)
		--                                          ^      ^       ^
		--                                      the value  |    is this entry
		--                                          a reader     still in the
		--                                          for its key  input table?
		return string.upper(value)
	end)

	print(#labels()) --> 2
end)

destroy()
```

`key` is a **reader**, not a number, because an entry's key changes when the list reorders — the entry for "Shield" moves from 2 to 1 when "Sword" is removed, and reading `key()` gives you where it is now.

> **The output is unordered**
>
> `for_values` keeps its results in a table keyed by the input **value** and builds the array by iterating that hash, so the order you get back is neither the order you put in nor stable between runs. If order matters — and in a list of UI rows it always does — drive `LayoutOrder` from `key()` inside the transform and let a `UIListLayout` sort them, or sort the array yourself before you use it.

**Ordering by the key, not by the array**

```luau
local destroy = fluid.root(function()
	local items = fluid.source({ "Sword", "Shield" })

	return fluid.create("Frame")({
		fluid.create("UIListLayout")({}),

		fluid.for_values(items, function(value, key, active)
			return fluid.create("TextLabel")({
				Text = value,
				LayoutOrder = key, -- a reader, so it follows the entry as it moves
			})
		end),
	})
end)

destroy()
```

`key` is already a function, so it can go straight into a property and stays correct when the list reorders.

> **`fluid.read` is a different tool**
>
> `fluid.read` really is one line — `if type(value) == "function" then value() else value` — and it is easy to assume it is what makes property tables work. It is not: `bind`, which is what `create` actually calls, never touches it. `read` is a **one-shot** unwrap with no subscription, used by `for_values`, `for_keys`, `show`, `switch`, `spring` and `lerp` to accept "a value or a reader" for their own arguments.

## for_keys, the other half

> **Duplicate values collapse into one entry**
>
> Every per-entry table — the key node, the scope, the output, the `active` node — is indexed by the value. Feed `for_values` a list with two identical values and you get **one** output for both: `{ "a", "a", "b" }` produces two rows, not three, and a `#tasks()` heading will disagree with what is on screen. For strings and numbers that can repeat, reach for `for_keys`.

`for_keys` produces one output per **key** instead of one per value, which makes it the tool to reach for whenever values can repeat. Its transform is not simply `for_values`' arguments swapped — which one is a reader flips as well.

|  | Argument 1 | Argument 2 | Argument 3 |
| --- | --- | --- | --- |
| `for_values` | `value` — the value itself | `key` — a **reader** | `active` |
| `for_keys` | `value` — a **reader** | `key` — the key itself | `active` |

The plain one and the reader one trade places. Everything else — `active`, the optional delay, the unordered output — is the same.

**One row per key**

```luau
local destroy = fluid.root(function()
	local scores = fluid.source({ Ada = 12, Grace = 12 })

	local rows = fluid.for_keys(scores, function(value, key, active)
		--                                  ^      ^
		--                             a reader   the key itself
		return fluid.create("TextLabel")({
			Text = function()
				return key .. ": " .. value()
			end,
		})
	end)

	print(#rows()) --> 2, even though both scores are 12
end)

destroy()
```

The same list through `for_values` would produce one row, because both values are `12`.

**Key points**

- `for_values(input, transform)` maps values; the transform gets `(value, key_reader, active)`.
- `key` is a reader because an entry's key moves when the list reorders — feed it straight into `LayoutOrder`.
- The output array is **unordered**, whichever of the two you use.
- `for_values` is keyed by the value, so duplicates collapse; `for_keys` is keyed by the key and does not.
- In `for_keys` the reader and the plain argument swap: `(value_reader, key, active)`.
