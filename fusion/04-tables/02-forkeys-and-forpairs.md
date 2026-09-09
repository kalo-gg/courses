# ForKeys and ForPairs

*The other two thirds of the family: rename the keys, or rewrite both halves at once.*

Fusion · Tables & Lists · 9 min

---

The three objects differ only in which half of the table they touch. Their source comments say it exactly:

| Object | Processes | Preserves |
| --- | --- | --- |
| `ForValues` | Values | Keys |
| `ForKeys` | Keys | Values |
| `ForPairs` | Keys and values together | Nothing — you return both |

**ForKeys: renaming the slots**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek, Children = Fusion.peek, Fusion.Children

local bySlot = scope:Value({ [1] = "Sword", [2] = "Shield" })

local byName = scope:ForKeys(bySlot, function(use, scope, slot)
	return "slot" .. slot
end)

-- { slot1 = "Sword", slot2 = "Shield" }
```

## When the key comes from the value

**ForPairs: when one depends on the other**

```luau
local scores = scope:Value({ Ada = 12, Grace = 30 })

local ranked = scope:ForPairs(scores, function(use, scope, name, score)
	return score, name -- swap them: score becomes the key
end)

-- { [12] = "Ada", [30] = "Grace" }
```

`ForPairs` returns two values, the new key and the new value.

`ForPairs` is the only one of the three that returns two values, and the only one where a mistake in the key half quietly costs you an entry. If the new key does not depend on the value — and usually it does not — `ForKeys` says so more clearly and gives Fusion more to reuse.

> **Keys have to stay unique**
>
> `ForKeys` and `ForPairs` can both produce a key that already exists — map every slot to `"item"` and you have one entry, not three. Fusion drops the duplicate and logs a **`forKeyCollision`** error naming the offending key, so this fails loudly rather than quietly. Prefer `ForValues` unless you genuinely need to change the keys.

One question decides it every time: which half of the table is different afterwards? Values only, and it is `ForValues` — which is what you want whenever the output is instances, because an instance is a value. Keys only, `ForKeys`. Both, with each depending on the other, `ForPairs`.

**Key points**

- `ForValues` processes values and keeps keys; `ForKeys` does the reverse; `ForPairs` returns both.
- All three take the same processor shape: `use`, a per-entry scope, then the data.
- A duplicated output key is dropped with a `forKeyCollision` error, not an exception.
- Reach for `ForValues` unless the keys genuinely have to change.
