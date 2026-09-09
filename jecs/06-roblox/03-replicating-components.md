# Replicating components

*Match component ids across realms, tag what replicates, record changes with signals and flush a per-frame diff.*

JECS · Roblox Integration · 11 min

---

Roblox replicates Instances for you, but it knows nothing about your components. If the client needs component *data*, you replicate it yourself: the server world is the truth and the client world is a **mirror** that only receives.

Three problems: both realms must agree on what an id refers to, the server must know what changed, and the client must apply it. This lesson and the next are a simplified version of the official `examples/networking` code.

## Component ids match; entity ids get mapped

**Component ids** are allocated in declaration order when `jecs.world()` runs. Because both realms require the same `components.luau`, `Position` is the same number on each side; change the order on one and the client writes health values into the position column.

**Entity ids** are a different story: the server allocates them and the client must not reuse them verbatim. The next lesson maps them.

## Marking what replicates

**server startup, before systems are created**

```luau
-- Networked is a tag from components.luau. Components are entities,
-- so a tag can be added to a component id like to anything else.
world:add(ct.Position, ct.Networked)
world:add(ct.Health, ct.Networked)
world:add(ct.Dead, ct.Networked) -- tags replicate too
```

The sending system finds every networked component with `world:each(ct.Networked)` and subscribes to its `added`, `changed` and `removed` signals. Each signal writes into a per-component **storage** keyed by entity: the latest value, or a sentinel meaning "removed". Once per frame the storages are flushed into a message; a new player first gets a **snapshot** built the same way.

**networking_send.luau: recording changes**

```luau
local function networking_send(world: jecs.World)
	local storages: { [jecs.Id]: { [jecs.Entity]: any } } = {}

	for component in world:each(ct.Networked) do
		local storage = {}
		storages[component] = storage

		if jecs.is_tag(world, component) then
			world:added(component, function(entity)
				storage[entity] = true
			end)
		else
			world:added(component, function(entity, _, value)
				storage[entity] = value
			end)
			world:changed(component, function(entity, _, value)
				storage[entity] = value
			end)
		end

		world:removed(component, function(entity)
			storage[entity] = "jecs.Remove" -- the entity lost this component
		end)
	end
```

**networking_send.luau: flushing the diff**

```luau
	return function()
		local diff = {}
		for component, storage in storages do
			local set, values, removed = {}, {}, {}
			local is_tag = jecs.is_tag(world, component)
			for entity, value in storage do
				if value ~= "jecs.Remove" then
					table.insert(set, entity)
					table.insert(values, if is_tag then 0 else value)
				elseif world:contains(entity) then
					table.insert(removed, entity)
				end
			end
			table.clear(storage)
			if #set > 0 or #removed > 0 then
				diff[tostring(component)] = { set = set, values = values, removed = removed }
			end
		end
		if next(diff) ~= nil then
			remotes.replication:FireAllClients(diff)
		end
	end
end
```

`remotes.replication` is a `RemoteEvent` named `replication` in `ReplicatedStorage`, as in the official `remotes.luau`.

| Field | Contents |
| --- | --- |
| message key | `tostring(component)`: Roblox remotes only carry arrays and string-keyed dictionaries reliably, so the numeric id becomes a string |
| `set` | entity ids that gained or changed the component this frame |
| `values` | parallel array of values; `0` is a placeholder for tags so the two arrays stay aligned |
| `removed` | entity ids that lost the component and are still alive |

The per-frame diff `{ set, values, removed }`; the snapshot is the same shape without `removed`.

Two details do quiet work. A storage keyed by entity means a component written many times in one frame produces **one** entry. And the `world:contains` check drops removals caused by deletion — the official example sends a `Destroy` tag for that.

> **Signals, not hooks, on networked components**
>
> Once a component is networked, never `world:set(C, jecs.OnAdd, …)` on it: that overwrites the signal dispatcher and replication silently stops for that component.

**Key points**

- Same `components.luau`, same order, same **component** ids on both realms.
- Tag components with `Networked`; find them with `world:each` and subscribe with `world:added/changed/removed` into per-component storages.
- `"jecs.Remove"` marks a removal; a `world:contains` check drops removals caused by deletion.
- New players get a snapshot; everyone gets a per-frame `{ set, values, removed }` diff keyed by `tostring(component)`.
