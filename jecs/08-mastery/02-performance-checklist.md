# The performance checklist

*What is cheap, what moves entities, and how to measure before optimizing.*

JECS · Best Practices & Capstone · 10 min

---

JECS is fast by default: the README's headline is iterating 800,000 entities at 60 frames per second, and more when you inline archetypes. Most performance problems in JECS games come from fighting the storage model rather than from JECS itself. Every rule below is one of the storage facts from Caching and archetypes turned into a habit. Here is the checklist, in order of impact.

| # | Rule | Why |
| --- | --- | --- |
| 1 | **Cache per-frame queries** once, in system setup. | A cached query iterates a prematched archetype list; an uncached one re-matches archetypes every call. |
| 2 | **Prefer `set` on existing components** over `add`/`remove` in hot loops. | Adds and removes move the entity between archetypes (copying its data). |
| 3 | **Batch structural changes** with `jecs.bulk_insert` / `jecs.bulk_remove`. | One archetype move instead of one per component. |
| 4 | **Watch relationship fragmentation.** | Every distinct pair target is its own archetype, plus wildcard indices. |
| 5 | **Keep hooks tiny.** | They run synchronously inside `set`/`remove`; heavy work there taxes every write. |
| 6 | **Call `world:cleanup()` occasionally.** | Frees empty archetypes; profile the interval. |
| 7 | **Inline with `query:archetypes()` only after measuring.** | The JECS docs put the iterator's call overhead at most of the iteration cost; inlining removes it but costs readability. |

Rules 1–3 are where nearly all real wins come from: they are about how often entities move between archetypes. Rules 4–7 matter only once a profiler points at them, which is why the checklist ends with *measure*.

**Batching component changes**

```luau
-- Three moves:
world:set(e, Position, Vector3.zero)
world:set(e, Velocity, Vector3.zero)
world:add(e, Enemy)

-- One move:
jecs.bulk_insert(world, e, { Position, Velocity, Enemy }, { Vector3.zero, Vector3.zero })
```

Tags come last in the id list with no matching value. That is a readability convention rather than a rule — `bulk_insert` looks up `values[i]` by the id's position, so `{ Position, Enemy, Velocity }` with `{ v, nil, v }` works too — but a dense values array is far easier to keep aligned by hand. Use this when spawning entities with many components.

## Inlining the hot loop

**Advanced: iterating archetype columns directly**

```luau
local moving = world:query(Position, Velocity):cached()

return function(dt: number)
	for _, archetype in moving:archetypes() do
		local entities = archetype.entities
		local positions = archetype.columns_map[Position]
		local velocities = archetype.columns_map[Velocity]
		for row = #entities, 1, -1 do
			positions[row] = positions[row] + velocities[row] * dt
		end
	end
end
```

This writes straight into the storage column, bypassing `set` (so no `OnChange` fires). Reserve it for measured hot spots.

Inlining is the one rule with a real cost. `query:archetypes()` hands you the storage itself, so you lose the loop variables, the safety of `world:set` and any hope that a later JECS release will change the layout under you. It buys perhaps a factor of two on a loop that is already the profiler's top row. Write the ordinary loop first, keep it in a comment beside the inlined one, and be able to say which frame time it bought.

> **Measure first**
>
> Use Jabby's scheduler applet or wrap systems with `os.clock()` before and after. The author's own advice is to run the benchmark yourself rather than trust numbers, including his.

**Key points**

- Cache per-frame queries; `set` existing components; batch structural changes with `bulk_insert`.
- Relationships fragment archetypes; hooks run synchronously; `cleanup()` frees empty archetypes.
- Inline via `query:archetypes()` only for measured hot spots.
