# Instances and entities

*Keep the Instance in its own component, let hooks own its lifetime, and find the entity behind a model.*

JECS · Roblox Integration · 10 min

---

The single most useful rule for JECS on Roblox is: **the world is the source of truth and the Instance is a view of it**. Gameplay data (`Position`, `Health`, `Owner`) lives in components. The `Model` that players actually see is *also* a component, but it is a separate one, so systems that do not care about rendering never have to touch it.

**components.luau (excerpt)**

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)

local Position = jecs.component() :: jecs.Id<Vector3>
local Model = jecs.component() :: jecs.Id<Model> -- the Roblox Instance lives here, alone

jecs.meta(Position, jecs.Name, "Position")
jecs.meta(Model, jecs.Name, "Model")

return {
	Position = Position,
	Model = Model,
}
```

`jecs.Id<Model>` names the Roblox *type* `Model`; the local `Model` is the component id. Luau keeps types and values in separate namespaces, so this is fine. Call the component `Renderable` if you find it clearer.

## Hooks that own the Instance

An Instance has a lifetime that must follow the component's: when the component appears, the model needs to know which entity it belongs to; when the component goes away, the model must be destroyed. That is exactly what hooks are for, and two timing facts make the pattern work:

- `OnAdd` runs **after** the value is stored and receives it as its third argument.
- `OnRemove` runs **before** the value is removed, so `world:get` still returns it inside the hook.

**startup (runs once, right after the world is created)**

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)
local ct = require(ReplicatedStorage.Shared.components)
local world = require(ReplicatedStorage.Shared.world)

-- Configure hooks BEFORE any entity gets a Model, or they never fire.
world:set(ct.Model, jecs.OnAdd, function(entity, id, model)
	-- Runs after the value is stored. Stamp the entity id on the Instance.
	model:SetAttribute("entityId", entity)
end)

world:set(ct.Model, jecs.OnRemove, function(entity, id, delete)
	-- Runs before the value is removed, so it is still readable here.
	local model = world:get(entity, ct.Model)
	if model then
		model:Destroy()
	end
end)

-- Later, anywhere in the codebase:
local zombie = world:entity()
world:set(zombie, ct.Position, Vector3.new(0, 5, 0))
world:set(zombie, ct.Model, model)
-- OnAdd already ran: model:GetAttribute("entityId") == zombie

world:delete(zombie) -- OnRemove destroyed the model for us
```

`model` here is a model cloned from `ReplicatedStorage.Assets` and parented to the workspace. Nothing outside the hooks ever calls `Destroy`: whether the entity is deleted or just loses its `Model`, the Instance goes with it.

> **What about the delete flag?**
>
> `OnRemove` receives `delete == true` when the whole entity is being deleted. Destroying an Instance is not a structural change to the world, so it is safe either way and this hook ignores the flag. Only hooks that would `add`, `remove` or `set` on the entity need to check `delete` and return early; better still, queue that work for a system.

## Rendering: copy data to the Instance

With the Instance in its own component, rendering is an ordinary system: query the data and the view together, and copy one into the other. Data flows in **one direction**, from the world to the Instance.

**systems/render.luau**

```luau
local ct = require(ReplicatedStorage.Shared.components)

local function render(world: jecs.World)
	local renderables = world:query(ct.Position, ct.Model):cached()

	return function()
		for _, position, model in renderables do
			model:PivotTo(CFrame.new(position))
		end
	end
end

return render
```

On the server this is already "networking": Roblox replicates the moved Instance to every client. Replicating components is for when you want to replicate the *component data* instead.

**Going the other way: Instance to entity.** Roblox hands you Instances all the time: `Touched` gives a `BasePart`, a raycast a hit part, a click a `Model`. Two answers, both maintained by the same hooks: the **attribute** stamped in `OnAdd`, which replicates to clients so they can read the server's entity id (Mirroring on the client lines those ids up), or a **plain table** `{ [Instance]: jecs.Entity }` filled in `OnAdd` and cleared in `OnRemove`.

> **Studio-placed things**
>
> Level designers tag Instances in Studio. A tiny system drains `CollectionService:GetInstanceAddedSignal("Enemy")` and `GetInstanceRemovedSignal` with `collect`, exactly like `PlayerAdded` in the next lesson, and spawns or deletes an entity per Instance; loop `CollectionService:GetTagged("Enemy")` once in setup for the ones placed before the system existed.

**Key points**

- Keep the Instance in its own component (`Model`/`Renderable`); gameplay data stays separate.
- `OnAdd` stamps `entityId` on the Instance; `OnRemove` reads it with `world:get` and destroys it. Configure both before first use.
- A render system copies world data to the Instance (`PivotTo`).
- One direction per component: the world writes to the Instance, or the Instance writes to the world, never both.
- Drain `CollectionService` signals with `collect` to spawn entities from tagged Instances, as the next lesson does for players.
