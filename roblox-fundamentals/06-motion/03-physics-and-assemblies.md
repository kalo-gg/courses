# Physics and assemblies

*Anchored parts stay where scripts put them; the rest are simulated in assemblies of welded parts, filtered by collision switches and groups.*

Roblox Fundamentals · Motion · 11 min

---

A mine cart is three parts: a body and two wheel blocks. Unanchor them, start a playtest, and they fall and tumble as three separate pieces, because physics moves each part on its own.

A part is either **anchored**, fixed wherever scripts put it, or **simulated**: gravity, collisions and constraints move it. Physics moves simulated parts in **assemblies**, parts joined rigidly that it treats as one body. Weld the cart's parts together and it moves as one cart.

## Welding an assembly

**A lamp as one assembly**

```luau
local lamp = workspace.Lamp

local function weld(a: BasePart, b: BasePart)
	local w = Instance.new("WeldConstraint")
	w.Part0 = a
	w.Part1 = b
	w.Parent = a
end

weld(lamp.Post, lamp.Shade)
weld(lamp.Post, lamp.Bulb)
```

A `WeldConstraint` locks two parts at their current offset, and they need not touch. It does nothing until it is in the Workspace. Weld each part to one main part, and never in a loop: the docs note that a cycle of welds can disable some of them.

Every part also carries a few switches that decide how it takes part in physics. They are per part, and the defaults are the ones a solid, touchable brick needs.

| Property | Default | What it decides |
| --- | --- | --- |
| `Anchored` | false | When true, physics never moves the part, and parts welded to it stay put too. |
| `CanCollide` | true | Whether other parts pass through it. Touch events still fire when it is off. |
| `CanTouch` | true | Whether `Touched` and `TouchEnded` fire. Both parts need it on. |
| `CanQuery` | true | Whether raycasts and spatial queries see it. It only counts when `CanCollide` is off. |
| `Massless` | false | When true, the part adds no mass to its assembly, as long as it is welded to a part with mass. |

## Collision groups

Sometimes parts should collide with the world but not with each other: a cart should roll along its track without shoving players off it. A **collision group** is a name. Register it on Workspace, say which pairs of groups collide, then put a part in a group by setting its `CollisionGroup`. Every part starts in `"Default"`, and every pair of groups collides until you say otherwise.

**Characters that pass through each other**

```luau
workspace:RegisterCollisionGroup("Players")
workspace:CollisionGroupSetCollidable("Players", "Players", false)

-- connect this to every player's CharacterAdded, as in the Players module
local function onCharacterAdded(character: Model)
	for _, part in character:GetDescendants() do
		if part:IsA("BasePart") then
			part.CollisionGroup = "Players"
		end
	end
end
```

The docs' own example: a group set not to collide with itself stops characters bumping into each other. `CollisionGroupSetCollidable` errors if either name is not registered. Registering costs more the more parts the place holds, so the docs recommend making groups at edit time, in Studio's Collision Groups editor, when you can.

> **Recognising older code**
>
> Older scripts manage groups through `PhysicsService`: `PhysicsService:RegisterCollisionGroup("Carts")`, the deprecated `CreateCollisionGroup` and `SetPartCollisionGroup`, and the numeric `CollisionGroupId`. The docs deprecate the service's group API in favour of groups on each world, and forward its calls to Workspace. Write `workspace:RegisterCollisionGroup` and set `part.CollisionGroup`.

**Key points**

- Anchored parts move only when a script moves them; every other part is simulated.
- An assembly is parts joined rigidly and moved by physics as one body; one anchored part anchors all of it.
- A `WeldConstraint` needs `Part0`, `Part1` and a place in the Workspace; weld to one main part, not in a loop.
- `CanCollide`, `CanTouch`, `CanQuery` and `Massless` are separate switches, and a non-colliding part still fires touches.
- Collision groups live on Workspace: `RegisterCollisionGroup`, `CollisionGroupSetCollidable` and `part.CollisionGroup`.
