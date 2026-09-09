# Component hooks

*OnAdd, OnChange and OnRemove: one callback per component, with precise timing rules.*

JECS · Hooks, Signals & Observers · 12 min

---

Component data often has to satisfy an invariant or trigger a side effect: a `Model` component should destroy its Instance when it goes away; a `Networked` component should register itself for replication. **Hooks** are component traits that run exactly when that component's data changes on any entity. Each component can have **one** `OnAdd`, **one** `OnChange` and **one** `OnRemove` hook, just like a type has one constructor and one destructor.

```luau
local Transform = world:component() :: jecs.Id<CFrame>

world:set(Transform, jecs.OnAdd, function(entity, id, data)
	-- Transform was added to entity with data (runs AFTER the value is stored)
end)

world:set(Transform, jecs.OnChange, function(entity, id, data)
	-- an existing Transform on entity was updated to data
end)

world:set(Transform, jecs.OnRemove, function(entity, id, delete)
	-- Transform is about to be removed (runs BEFORE removal, data still readable)
	-- delete == true when the whole entity is being deleted
end)
```

Hooks are attached with a plain `world:set` on the component id, using the built-in `jecs.OnAdd`, `jecs.OnChange` and `jecs.OnRemove` components. `id` is the component (or pair) that triggered the hook.

| Hook | Fires when | Timing | Callback |
| --- | --- | --- | --- |
| `OnAdd` | the component is added to an entity (first `set`, or `add`) | after the value is written | `(entity, id, data)` — `data` is `nil` for `world:add` |
| `OnChange` | `set` updates a component the entity **already has** | after the new value is written | `(entity, id, data)` |
| `OnRemove` | `remove`, `clear`, or `delete` takes the component away | before removal; `world:get` still works | `(entity, id, delete)` — `delete` is `true` during `world:delete` |

The three hooks are not symmetric, and the asymmetry is the whole trick. `OnAdd` runs **after** the value is in the column, so `world:get` already answers; it fires on the first `set` only, never on later updates. `OnChange` is the mirror: updates only, never the first `set`. `OnRemove` runs **before** the data leaves, which is the only reason a destructor hook can read what it is about to destroy.

## Two rules that bite

> **Rule 1: define hooks before the component is used**
>
> JECS reads a component's hooks when it first creates storage for that component, which happens the first time it is added to any entity. Set hooks immediately after creating the component. If you attach them later, they will simply never run.

> **Rule 2: check `delete` in OnRemove and do not restructure**
>
> During `world:delete` every `OnRemove` fires with `delete == true`. Since JECS 0.11.0, adding or removing components inside `OnRemove` is unsupported: two paths (replacing an `Exclusive` target, and `jecs.bulk_remove`) hard-error with *"on_remove must not perform structural changes"*, and the rest — including `world:delete` — let it happen silently while the entity is mid-teardown. Check `delete` and return early; if follow-up structural changes are needed, queue them for a system.

**The canonical Model component (from the JECS examples)**

```luau
local Model = world:component() :: jecs.Id<Model>

world:set(Model, jecs.OnAdd, function(entity, id, model)
	-- fires once, after the model is stored; handy for back-references
	model:SetAttribute("entityId", entity)
end)

world:set(Model, jecs.OnRemove, function(entity, id, delete)
	-- OnRemove runs before removal, so the data is still readable
	local model = world:get(entity, Model)
	if model then
		model:Destroy()
	end
end)
```

Destroying an Instance is *not* a structural change to the world, so it is safe whether `delete` is true or false.

**Bailing out during deletion**

```luau
world:set(Health, jecs.OnRemove, function(entity, id, delete)
	if delete then
		return -- the entity is going away; nothing to tidy
	end
	-- normal removal: queue follow-up work for a system to apply
	table.insert(pending_deaths, entity)
end)
```

Hooks are single-owner by design: `world:set(Model, jecs.OnRemove, fn)` overwrites whatever was there, silently. That is fine for the one piece of code that owns a component's lifetime — the module that declared it — and wrong for everything else. When a second system also wants to know, it subscribes with a signal instead, which is the next lesson.

> **Debug mode will not catch this for you**
>
> It is tempting to assume `jecs.world(true)` guards the hook rules. It does not. DEBUG wraps `add`, `set` and `remove` — stale handles, unregistered ids, wildcard misuse — and `world:delete` is bound before that wrapper is installed and never rebound. Add a `Dead` tag from an `OnRemove` during a `world:delete` and it goes through silently, DEBUG or not.
>
> JECS 0.11 does hard-error on structural changes in `OnRemove`, but only on two paths: replacing the target of an `Exclusive` relationship, and `jecs.bulk_remove`. Both raise *"on_remove must not perform structural changes"*. Plain `world:remove` and `world:delete` allow them quietly, which is worse — the same mistake either shouts or corrupts depending on which call reached the hook.
>
> So the rule in this lesson is one you have to keep yourself.

**Key points**

- One `OnAdd`, `OnChange`, `OnRemove` per component, attached with `world:set(Component, jecs.OnX, fn)`.
- `OnAdd` runs after the value is stored (first add only); `OnChange` only for updates; `OnRemove` before removal.
- Attach hooks right after creating the component, before it is used anywhere.
- In `OnRemove`, check the `delete` flag and never make structural changes; queue follow-ups instead.
