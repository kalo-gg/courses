# Testing outside Roblox

*Because JECS is pure Luau, world logic can be tested from the command line.*

JECS · Best Practices & Capstone · 8 min

---

JECS has zero Roblox dependencies. Its own test suite runs with the Luau command-line interpreter (`luau test/tests.luau`), and the repository ships a tiny harness, `modules/testkit.luau`, exposing `TEST`, `CASE` and `CHECK` helpers. You can do the same for your game: keep systems free of Roblox globals and run them under the `luau` CLI or Lune. The two Roblox Integration projects already follow the rule: their system files take `Players`, `collect` and `CFrame` as parameters, which is what lets them run in this sandbox and under `luau` alike.

## A world is all you need

**A system that is easy to test**

```luau
-- systems/damage.luau: no Roblox services, everything comes in as parameters
local function damage(world: jecs.World, ct)
	local hits = world:query(ct.Health, ct.IncomingDamage):cached()
	return function()
		for entity, health, incoming in hits do
			world:set(entity, ct.Health, health - incoming)
			world:remove(entity, ct.IncomingDamage)
		end
	end
end

return damage
```

**tests/damage.luau (run with: luau tests/damage.luau)**

```luau
local jecs = require("@jecs")
local damage = require("../systems/damage")

local ct = {
	Health = jecs.component() :: jecs.Id<number>,
	IncomingDamage = jecs.component() :: jecs.Id<number>,
}
local world = jecs.world()
local run = damage(world, ct)

local e = world:entity()
world:set(e, ct.Health, 100)
world:set(e, ct.IncomingDamage, 30)

run()

assert(world:get(e, ct.Health) == 70, "health should drop to 70")
assert(not world:has(e, ct.IncomingDamage), "damage should be consumed")
print("damage system: ok")
```

`@jecs` is an alias defined in `.luaurc`. Outside Roblox use `vector.create` instead of `Vector3.new`, or inject a vector constructor as a parameter.

Read the two blocks together and the rule falls out: a system takes the world and whatever else it needs **as arguments**, and returns a function. Nothing inside it names a service, a global or a singleton it did not receive. A test then builds a world, calls the factory with plain tables, runs the returned function as many times as it likes and asserts on `world:get`. No Studio, no `wait`, no frame.

> **This site is a test runner too**
>
> Every exercise here runs your code in a WebAssembly build of Luau against the real JECS source. The `assert`-based checks you see in the final project are exactly the style you can use in a `tests/` folder.

## Faking the parts Roblox owns

Some systems genuinely need `Players`, a `RemoteEvent` or `CFrame`. Inject them and the fake is a table. `Players` becomes `{ PlayerAdded = signal(), PlayerRemoving = signal() }` where `signal()` is ten lines holding a list of listeners and a `Fire`; `collect` is the same `collect` the game uses, since it only needs `:Connect`; `CFrame` becomes `{ new = function(p) return { Position = p } end }` when all the system does is `PivotTo`. The test then *drives* the fake: fire `PlayerAdded`, run the system, assert an entity exists.

**Testing the players system with a fake Players**

```luau
local function signal()
	local listeners = {}
	return {
		Connect = function(_, fn) table.insert(listeners, fn) end,
		Fire = function(_, ...) for _, fn in listeners do fn(...) end end,
	}
end

local function count()
	local n = 0
	for _ in world:query(ct.Player) do n += 1 end
	return n
end

local Players = { PlayerAdded = signal(), PlayerRemoving = signal() }
local run = players(world, Players, collect)
local ryan = { Name = "Ryan" }

Players.PlayerAdded:Fire(ryan)
run()
assert(count() == 1, "the join should create one entity")

Players.PlayerRemoving:Fire(ryan)
run()
assert(count() == 0, "the leave should delete it")
```

The same file Project 2 shipped, tested with no DataModel anywhere. Fire the *same* table for the join and the leave, exactly as Project 2's `main.luau` does: the system keys its `by_player` map on the player value it was handed, so a second `{ Name = "Ryan" }` would be a different player.

**Key points**

- JECS runs under the `luau` CLI and Lune; so can your systems if they avoid Roblox globals.
- Inject the world and dependencies; assert on the world after running a system.
- The JECS repo's `testkit` shows the author's own harness.
