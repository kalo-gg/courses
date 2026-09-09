# Observers and monitors

*React when an entity starts or stops matching a whole query, using the OB module shipped with JECS.*

JECS · Hooks, Signals & Observers · 8 min

---

Hooks and signals watch a *single* component. Often you want to know when an entity starts matching a **combination**: "has `PivotTo` *and* `Renderable`". The JECS repository ships a small copy-in module for that, `modules/OB/module.luau`, built on top of signals. Copy it into your project (for example as `ReplicatedStorage/Shared/OB`) and require it. Both take a plain `world:query(...)` and cache a private copy of it, so pass an uncached query.

**OB.observer**

```luau
local OB = require(ReplicatedStorage.Shared.OB)

local function pivots_added(entity: jecs.Entity)
	local dst, model = world:get(entity, ct.PivotTo, ct.Renderable)
	if dst == nil or model == nil then
		return
	end
	world:set(entity, ct.Transform, dst)
	world:remove(entity, ct.PivotTo)
	;(model :: Model):PivotTo(dst)
end

-- Fires when an entity begins matching the query, or changes while matching.
OB.observer(world:query(ct.PivotTo, ct.Renderable), pivots_added)
```

From `modules/OB/examples/example.luau`. The callback receives just the entity; read what you need with `world:get`. The leading `;` on the last line stops Luau from parsing it as a call on the previous line.

**OB.monitor**

```luau
local monitor = OB.monitor(world:query(ct.Position, ct.Renderable))

monitor.added(function(entity)
	-- entity now has both Position and Renderable
end)

monitor.removed(function(entity)
	-- entity stopped matching (lost one of them, or was deleted)
end)

-- later:
monitor.disconnect()
```

An **observer** fires while an entity is entering the match: it runs once per id that completes the combination, so an entity that gains two of the query's terms in two calls is reported twice. A **monitor** fires on the transition itself — once when the entity starts matching and once when it stops — which is what you want for a counter, a spawn or a teardown.

## Choosing between the four tools

| Tool | Watches | Use when |
| --- | --- | --- |
| Hook | one component, one callback | enforcing an invariant for that component (destroy Instance, validate data) |
| Signal | one component, many callbacks | several systems react to the same component event |
| `OB.observer` | a query (all terms present), fires on enter and on change | "do X once these components are all there" |
| `OB.monitor` | a query, separate enter/leave callbacks | tracking membership of a set (spawn/despawn visuals, counters) |
| Previous-pair queries (Change tracking with queries) | a component, batched per frame | frame-ordered processing without callbacks |

Four layers, narrowing as you go down: hooks own one component's lifetime, signals let many listeners watch one component, observers and monitors watch a *combination*, and a previous-pair query answers "what changed since last frame?" in the frame's own order. Pick the narrowest one that answers your question.

> **Pass an uncached query**
>
> The OB module takes a plain `world:query(...)` (with any `with`/`without` filters) and caches it internally. Do not pass an already-cached query.

**Key points**

- `modules/OB` (copy-in) provides `OB.observer(query, fn)` and `OB.monitor(query)` with `added`/`removed`/`disconnect`.
- Observers fire when an entity enters or changes while matching a whole query.
- Choose hook, signal, observer or Previous-pair queries by scope and timing needs.
