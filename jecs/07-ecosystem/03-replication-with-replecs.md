# Replication with replecs

*Mark components and entities for replication, pick reliable or unreliable channels, and carry the buffers.*

JECS · The JECS Ecosystem · 12 min

---

Roblox Integration built replication by hand, with no relevancy filtering, bandwidth optimization or authority checks. **replecs** by PepeElToro41 packages all of that for JECS: transport-agnostic (it produces `buffer`s; you send them), per-entity player filters, per-component channels, relationship pairs and id remapping. Server to client only.

| Version | Released | Use it when… |
| --- | --- | --- |
| `0.3.1` | February 2026 | Current stable. wally `pepeeltoro41/replecs@0.3.1`, npm `@rbxts/replecs@0.3.1`. |
| `0.4.0-rc.3` | September 2026 | You are on **JECS 0.11**, which this release-candidate line targets. wally `pepeeltoro41/replecs@0.4.0-rc.3`, npm dist-tag `next`. (`rc.2`, from 30 August, is the previous one.) |

Which one you need depends on the JECS version in your lockfile. Open `wally.lock` (or `package-lock.json`), find the JECS line, and pick the replecs release that matches before anything else.

> **Require replecs before creating the world**
>
> replecs *preregisters* its own components and tags (`replecs.shared`, `replecs.networked`, the channel relations). Like any preregistered id (Tags, singletons and preregistration) they only exist in worlds created **after** the module was required. `require(replecs)` must therefore run before `jecs.world()`, on both the server and the client. Get this wrong and nothing replicates, usually without a helpful error.

## Marking components as shared

replecs replicates a component only if you tell it to. Two things are required, on **both** server and client:

- add the `replecs.shared` tag to the component, and
- give it a `jecs.Name`.

The name is how replecs matches a component on the server to the same component on the client, so it must be identical on both sides. Because component declarations live in a shared module, both requirements are met in one place.

**shared/components.luau**

```luau
local Health = world:component() :: jecs.Id<number>
world:add(Health, replecs.shared)
world:set(Health, jecs.Name, "Health")

local Position = world:component() :: jecs.Id<Vector3>
world:add(Position, replecs.shared)
world:set(Position, jecs.Name, "Position")

local Targeting = world:entity() -- a relation we also want replicated
world:add(Targeting, replecs.shared)
world:set(Targeting, jecs.Name, "Targeting")

return { Health = Health, Position = Position, Targeting = Targeting }
```

Only `shared` components are ever serialized. Everything else (server-only AI state, client-only render handles) stays local without any extra work.

**Networked entities and channels.** Marking components says *what can* replicate. Marking entities says *which entities* do. Add `replecs.networked` to an entity, then say, per component, which channel it should use:

**server: spawning a replicated NPC**

```luau
local pair = jecs.pair
local ct = require(ReplicatedStorage.Shared.components)

local npc = world:entity()
world:set(npc, ct.Health, 100)
world:set(npc, ct.Position, Vector3.new(0, 5, 0))
world:add(npc, pair(ct.Targeting, player_entity))

world:add(npc, replecs.networked)                        -- replicate this entity to everyone
world:add(npc, pair(replecs.reliable, ct.Health))        -- Health: never dropped, in order
world:add(npc, pair(replecs.unreliable, ct.Position))    -- Position: cheap, may be dropped
world:add(npc, pair(replecs.relation, ct.Targeting))     -- replicate the (Targeting, *) pairs
```

To limit an entity to certain players, `world:set(npc, replecs.networked, filter)` takes a filter value instead of the bare tag. The filter's shape is version-specific, so look it up in the replecs docs for the release you installed.

**server/replication.server.luau**

```luau
local replicator = replecs.create_server(world)
replicator:init()

-- A new client asks for the full state once
remotes.receive_full:set_callback(function(player)
	replicator:mark_player_ready(player)
	return replicator:get_full(player)
end)

-- A system run at ~20 Hz, e.g. with Planck.timePassed(1 / 20)
local function replicate()
	for player, buf, variants in replicator:collect_updates() do
		remotes.send_updates:fire(player, buf, variants)
	end
end

return replicate
```

Server, client and the wire: `remotes` stands in for whatever transport you pick. `get_full` returns the initial snapshot for one player, `mark_player_ready` starts including that player in updates, and `collect_updates` yields one `buffer` (plus a `variants` table) per player.

The client mirrors it: `create_client(world)`, `init()`, `apply_full(buf, variants)` once, then `apply_updates` for each incoming update, drained **inside a system**. What carries `buf`? Any buffer-based networking library: **Zap** and **Blink** compile an IDL into typed modules and both offer a *polling* call mode (Blink's `Iter()`, Zap's since 0.6.18) that queues events for a system to drain; **ByteNet** is runtime-defined but callbacks-only and in maintenance mode. Versions are on the reference page.

> **Alternatives**
>
> - **feces** (pesde `killergg/feces`, 1.3.0): smaller than replecs, but you must create replicated components in the same order on every world.
> - **duplecs** (wally `demi-dog/duplecs`, 1.0.0, August 2026): new and small, worth watching.
> - **Hand-rolled** (Roblox Integration): full control and no dependency, but relevancy, bandwidth and authority checks are all yours to write.

**Key points**

- replecs is transport-agnostic server → client replication for JECS; 0.3.1 is stable, the 0.4.0 release candidates (rc.3 at the time of writing) target JECS 0.11. Match your lockfile.
- `require(replecs)` **before** `jecs.world()`, on both sides.
- Components replicate only with `replecs.shared` plus an identical `jecs.Name` on server and client.
- Entities replicate only with `replecs.networked`; choose `pair(replecs.reliable, C)`, `pair(replecs.unreliable, C)` or `pair(replecs.relation, Rel)` per component.
- Client: `create_client` → `init` → `apply_full` → `apply_updates` inside a system; carry the buffers with Zap or Blink's polling events.
