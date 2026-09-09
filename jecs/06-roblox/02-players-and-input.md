# Players and input as data

*Give every player an entity, spawn their character into it, and turn input into components a system consumes.*

JECS · Roblox Integration · 11 min

---

Players are entities too. The `Player` Instance goes in one component, the character `Model` in another, and a reverse map lets Roblox callbacks find the entity. `Player`, `Character`, `MoveDirection` and a client-only `LocalPlayer` singleton are all preregistered in `components.luau`. Systems and the frame sketched this system; here it grows into the real thing, including characters, and then the same idea is applied to **input**.

**server/systems/players.luau**

```luau
local function players(world: jecs.World)
	local players_added = collect(Players.PlayerAdded)
	local players_removing = collect(Players.PlayerRemoving)
	local by_player: { [Player]: jecs.Entity } = {}

	return function()
		for player in players_added do
			local e = world:entity()
			world:set(e, ct.Player, player)
			world:set(e, jecs.Name, player.Name)
			by_player[player] = e
			if player.Character then
				world:set(e, ct.Character, player.Character)
			end
		end

		for player in players_removing do
			local e = by_player[player]
			if e then
				world:delete(e)
				by_player[player] = nil
			end
		end
	end
end
```

Left out for room: a `characters_added` table that holds one `collect(player.CharacterAdded)` iterator per player, drained in a loop between these two so later characters land in `ct.Character` as well. `collect` returns the iterator *and* the connection, so a parallel `connections` table lets the `PlayerRemoving` drain disconnect it as the entity goes.

Every Roblox event is queued and processed here, in system order, once per frame. Nothing else in the codebase spawns or deletes a player entity, so there is one place to look when a player is missing. Note the `player.Character` check: `PlayerAdded` is drained a frame later, so the first `CharacterAdded` may already have fired.

> **Ownership through relationships**
>
> Entities that belong to a player (their pet, their placed blocks) should carry `pair(OwnedBy, player_entity)` rather than a copy of the `Player` Instance. With `pair(jecs.OnDeleteTarget, jecs.Remove)` on `OwnedBy` (Cleanup traits and Exclusive) the loot survives and simply loses its owner when the player leaves; switch the action to `jecs.Delete` only for things that should not outlive the player, such as their placed blocks.

## Input as data (client)

On the client the local player gets an entity as well, and its id is stored in the `LocalPlayer` **singleton** (a component set on itself, Tags, singletons and preregistration). Instead of moving the character from an input callback, an input system writes what the player *wants* into `MoveDirection` every frame, and a movement system decides what to do with it.

**client/systems/local_player.luau**

```luau
local function local_player(world: jecs.World)
	local player = Players.LocalPlayer
	local characters_added = collect(player.CharacterAdded)

	local me = world:entity()
	world:set(me, ct.Player, player)
	world:set(ct.LocalPlayer, ct.LocalPlayer, me) -- the singleton

	return function()
		for character in characters_added do
			world:set(me, ct.Character, character)
		end
	end
end
```

Anywhere on the client, `world:get(ct.LocalPlayer, ct.LocalPlayer)` answers "which entity is me?" without a global variable.

**client/systems/input.luau**

```luau
local function input(world: jecs.World)
	local held: { [Enum.KeyCode]: boolean } = {}

	-- The callback only records state. It never touches the world.
	local function on_move(_, state: Enum.UserInputState, object: InputObject)
		held[object.KeyCode] = state == Enum.UserInputState.Begin
		return Enum.ContextActionResult.Pass
	end

	ContextActionService:BindAction("ecs_move", on_move, false,
		Enum.KeyCode.W, Enum.KeyCode.A, Enum.KeyCode.S, Enum.KeyCode.D)

	return function()
		local me = world:get(ct.LocalPlayer, ct.LocalPlayer)
		if not me then
			return
		end

		local direction = Vector3.zero
		if held[Enum.KeyCode.W] then direction += Vector3.new(0, 0, -1) end
		if held[Enum.KeyCode.S] then direction += Vector3.new(0, 0, 1) end

		world:set(me, ct.MoveDirection, direction)
	end
end
```

A and D are the same shape on the X axis. The callback pushes into a local table and the system pulls from it, exactly like `collect`.

**client/systems/movement.luau**

```luau
local function movement(world: jecs.World)
	local movers = world:query(ct.MoveDirection, ct.Character):without(ct.Stunned):cached()

	return function()
		for _, direction, character in movers do
			local humanoid = character:FindFirstChildOfClass("Humanoid")
			if humanoid then
				humanoid:Move(direction, true) -- relative to the camera
			end
		end
	end
end
```

This is where the pay-off shows. The input system knows nothing about stuns, cutscenes or vehicles; the movement system just filters `:without(ct.Stunned)`. A bot needs to walk? An AI system sets `MoveDirection` on the bot's entity and the same movement system moves it. Replays and automated tests record those values and play them back.

> **Two realms, two loops**
>
> The server runs its whole list from `RunService.Heartbeat`. The client keeps two lists: simulation (`local_player`, `input`, `movement`) on `Heartbeat`, and camera or interpolation work on `PreRender`, which fires right before the frame is drawn (`RenderStepped` is its deprecated alias). The server has no `PreRender`.

**Key points**

- Players are entities: `Player` and `Character` components plus a `by_player` reverse map.
- Drain `PlayerAdded`, `PlayerRemoving` and `CharacterAdded` with `collect`; disconnect per-player signals on leave; the client adds a `PreRender` list for camera work.
- The client stores its own entity in the `LocalPlayer` singleton: `world:set(ct.LocalPlayer, ct.LocalPlayer, me)`.
- Input is data: a callback records, an input system writes `MoveDirection`, a movement system consumes it.
- Intent-as-data means bots, replays and status effects need no changes to the input code.
