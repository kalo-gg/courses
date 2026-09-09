# Mirroring on the client

*Map server ids to local entities, apply set and removed, and learn why the client never adopts a server id.*

JECS · Roblox Integration · 11 min

---

The client world only ever *receives*. Each message names entities by the **server's** ids, so the first job is translation: a table from server id to a locally allocated entity, filled the first time an id is seen — the official `networking_recv.luau` calls it `ecs_ensure_entity`. After that, applying a diff is the ordinary API: `set` for values, `add` for tags, `remove` for the `removed` list.

**ensure_entity**

```luau
-- server id -> local id. This is the official example's client_ids.
local client_ids: { [jecs.Entity]: jecs.Entity } = {}

local function ensure_entity(server_entity: jecs.Entity): jecs.Entity
	local local_entity = client_ids[server_entity]
	if not local_entity then
		local_entity = world:entity() -- a normal, locally allocated id
		client_ids[server_entity] = local_entity
	end
	return local_entity
end
```

## Why not mirror the ids 1:1?

> **Why not mirror the server's ids 1:1?**
>
> It is the obvious design, the JECS changelog advertises it ("`World:range` … makes it possible to receive ids from a server that don't have to be mapped to local ids"), and on **0.11.0 it does not work**.
>
> `world:range(1000, 5000)` on the client and `world:entity(server_id)` for a mirrored entity look fine together — until the client allocates one of its own. `world:entity(id)` *raises* the allocator's `max_id` to whatever id you passed, and the plain allocator then hands out `max_id + 1` and asserts it is below `range_end`. So one mirrored id of 5001 pushes `max_id` past the client's own ceiling and the next `world:entity()` throws:
>
> `Entity is outside range`
>
> That is guaranteed in any real session, because clients create their own entities all the time — VFX, UI, prediction — which is exactly what the 1000–4999 range was for. Map the ids instead. It costs one table.

The map has one more job than translation: it is the client's record of what it has been told about. An id that is not in it has never arrived, which is what makes an out-of-order or duplicated message harmless — the client either allocates on first sight or does nothing at all. Clear a row when you delete its entity, or the next message with that server id will find a dead handle.

## Applying a message

**networking_recv.luau: the per-frame body**

```luau
	return function()
		for message in messages do
			for key, map in message do
				local component = tonumber(key) :: jecs.Id -- same id as on the server
				local is_tag = jecs.is_tag(world, component)

				if map.set then
					for i, server_entity in map.set do
						local entity = ensure_entity(server_entity)
						if is_tag then world:add(entity, component)
						else world:set(entity, component, map.values[i]) end
					end
				end

				if map.removed then
					for _, server_entity in map.removed do
						local entity = client_ids[server_entity]
						if entity and world:contains(entity) then
							world:remove(entity, component)
						end
					end
				end
			end
		end
	end
```

Note the asymmetry: `set` creates entities, `removed` never does — an id the client has never heard of has nothing to remove.

The client allocates every id it uses with a plain `world:entity()`, so it stays inside its own allocator and never collides with anything; the `client_ids` table is the only place the server's numbering exists. This simplified client also never deletes mirrored entities; the official example replicates a `Destroy` tag and a client system deletes anything carrying it — and when it does, it must also clear that entity's row out of `client_ids`, or the map grows forever and a recycled server id resolves to a dead local one.

> **What this does not do**
>
> The official docs are explicit that this is a foundation, not a product. It has **no bandwidth optimization** (values go as plain tables at frame rate), **no relevancy** (every client gets everything), **no interpolation** (mirrored positions jump), and **no authority validation** (it is server to client only; never apply client-sent snapshots to the server world). Replication with replecs solves most of this on top of JECS, and Zap or Blink carry it efficiently.

**Key points**

- The client allocates every id it uses with `world:entity()`; `client_ids` is the only place the server's numbering exists.
- `set` creates entities through `ensure_entity`; `removed` never does.
- `world:entity(server_id)` plus `world:range` breaks the client's allocator on 0.11.0: map instead.
- Deletion travels as a replicated `Destroy` tag; clear the entity's row from `client_ids` when you delete it.
- No bandwidth optimization, relevancy, interpolation or authority validation: see replecs, Zap and Blink.
