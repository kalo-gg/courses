# Hierarchies with ChildOf

*The built-in parent/child relationship, walking up and down the tree, and cascading deletes.*

JECS · Relationships · 9 min

---

JECS ships a built-in relationship, `jecs.ChildOf`, for parent/child trees. It is pre-configured with two traits you will meet in the last lesson of this module: it is **exclusive** (an entity has at most one parent) and deleting a parent **deletes its children**.

```luau
local pair = jecs.pair
local ChildOf = jecs.ChildOf

local sun = world:entity()
local earth = world:entity()
local moon = world:entity()

world:add(earth, pair(ChildOf, sun))
world:add(moon, pair(ChildOf, earth))

print(world:parent(moon) == earth)              -- true
print(world:has(moon, pair(ChildOf, earth)))    -- true

for child in world:children(sun) do              -- earth
	print(child)
end

-- equivalent, and combinable with other terms:
for child in world:query(pair(ChildOf, sun)) do end
```

`world:parent(e)` is shorthand for the first `ChildOf` target. `world:children(p)` iterates entities with `pair(ChildOf, p)`.

## Walking the tree

Two calls read the tree. **`world:parent(entity)`** returns the one entity on the other end of `ChildOf`, or `nil` at the root — `ChildOf` is exclusive, so there is never a second answer. **`world:children(entity)`** iterates the other direction. Neither is a query: they read the relationship index directly, which is why walking a transform chain every frame is cheap.

**Accumulating transforms down a hierarchy (from the JECS examples)**

```luau
local Name = world:component() :: jecs.Id<string>

local function path(entity)
	local str = world:get(entity, Name)
	local parent = world:parent(entity)
	while parent do
		str = world:get(parent, Name) .. "/" .. str
		parent = world:parent(parent)
	end
	return str
end

local function iterate(entity, parent_position)
	local local_position = assert(world:get(entity, Position))
	local world_position = local_position + parent_position
	print(path(entity), world_position)

	for child in world:query(pair(ChildOf, entity)) do
		iterate(child, world_position)
	end
end

iterate(sun, Vector3.zero)
```

Querying `pair(ChildOf, entity)` for a runtime `entity` is the textbook ad-hoc (uncached) query.

## Cascading deletion

`ChildOf` ships with `pair(jecs.OnDeleteTarget, jecs.Delete)` already configured, which is the trait *Cleanup traits* takes apart at the end of this module. The consequence is the useful part: deleting a parent deletes its whole subtree in one call, and no system ever sees a child pointing at an entity that is gone.

```luau
world:delete(sun)

print(world:contains(earth), world:contains(moon)) -- false  false
```

Children are deleted before parents, and their hooks fire first (as long as the graph has no cycles). Because deletion cascades, a wave of enemies parented to a wave entity disappears with one `world:delete` — Project 1 uses exactly this.

> **Use ChildOf for real trees, not for everything**
>
> Model hierarchies, UI trees and attachment points are natural `ChildOf` uses. For "belongs to a player" or "is in this inventory" prefer your own relationship (`OwnedBy`, `InInventory`) so you control the cleanup policy rather than inheriting cascading deletes.

**Key points**

- `world:add(child, pair(jecs.ChildOf, parent))` builds hierarchies.
- `world:parent(e)` and `world:children(p)` navigate; `query(pair(ChildOf, p))` combines with other terms.
- ChildOf is exclusive (one parent) and deleting a parent deletes its subtree, children first.
