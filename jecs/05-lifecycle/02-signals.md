# Signals: many listeners

*world:added, world:changed and world:removed with unsubscribe functions.*

JECS · Hooks, Signals & Observers · 9 min

---

Hooks are single-owner: exactly one callback per event per component. When several independent systems want to react to the same event (replication *and* audio *and* UI all care about `Health` changes), you want **signals**. Since JECS 0.9 they are part of the core world API and support any number of subscribers.

```luau
local Position = world:component() :: jecs.Id<{ x: number, y: number }>

local unsub_added = world:added(Position, function(entity, id, value)
	print(`Position added to {entity}: ({value.x}, {value.y})`)
end)

local unsub_changed = world:changed(Position, function(entity, id, value)
	print(`Position changed on {entity}: ({value.x}, {value.y})`)
end)

local unsub_removed = world:removed(Position, function(entity, id, delete)
	if delete then
		print(`{entity} deleted (had Position)`)
	else
		print(`Position removed from {entity}`)
	end
end)

local e = world:entity()
world:set(e, Position, { x = 10, y = 20 })  -- added
world:set(e, Position, { x = 30, y = 40 })  -- changed
world:remove(e, Position)                   -- removed

unsub_added() -- stop listening
```

Each subscription returns a function; call it to unsubscribe. Calling it a second time errors, so keep track of what you have disconnected.

| Signal | Callback |
| --- | --- |
| `world:added(C, fn)` | `(entity, id, value, oldarchetype)` |
| `world:changed(C, fn)` | `(entity, id, value, oldarchetype)` |
| `world:removed(C, fn)` | `(entity, id, delete)` |

The same timing rules as hooks apply: `added` after the write, `removed` before removal with the `delete` flag.

**Signals on a relationship**

```luau
local OwnedBy = world:entity()

world:added(OwnedBy, function(entity, id, value)
	print(`{entity} now owned via {id}`)
end)

world:add(sword, pair(OwnedBy, player)) -- fires: id is that concrete pair
```

> **The fourth argument: `oldarchetype`**
>
> `added` and `changed` are dispatched with a fourth value that is easy to overlook: the archetype the entity was in **before** the write. Most listeners ignore it, but it is the only way to answer *did this entity just enter my set, or was it already in it?*
>
> A query matches a set of archetypes. If `oldarchetype` is already one of them, this write was an update to something you were watching. If it is not, the entity just moved in. That is exactly how the bundled observer module (`OB`) tells "entered the query" apart from "changed while already matching" — you cannot write your own monitor without it.

## When a signal is the right tool

Reach for a signal whenever the answer to "who else cares?" is more than one, or when the code that cares does not own the component. Replication, a HUD counter and an audio cue can all watch `Health` at once, and each holds its own unsubscribe function. Reach for a hook when exactly one piece of code owns the component's lifetime and must run before the data disappears — the `Model` destructor from the previous lesson.

> **Signals use the hook slot under the hood**
>
> The first `world:added(C, …)` installs a dispatcher as `C`'s `OnAdd` hook (an existing hook is kept as the first listener). If you later call `world:set(C, jecs.OnAdd, fn)` yourself, you overwrite that dispatcher and silently break every signal listener. Pick one mechanism per component, or attach the hook first and signals afterwards.

Signals also work on **relationships**: subscribing to `world:added(OwnedBy, fn)` fires for every `pair(OwnedBy, target)`, with `id` telling you which concrete pair was added. The official networking example uses exactly this to track replicated pairs.

**Key points**

- `world:added`, `world:changed`, `world:removed` accept many listeners and return unsubscribe functions.
- Same timing and `delete` semantics as hooks.
- Do not set a raw hook on a component after subscribing signals on it.
- Signals on relationships fire for every pair; `id` tells you which one.
