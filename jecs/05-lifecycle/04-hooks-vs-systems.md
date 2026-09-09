# Hooks are not systems

*Design guidance from the JECS author on where logic lives, and a refactor that moves gameplay out of a hook.*

JECS · Hooks, Signals & Observers · 9 min

---

The JECS repository ends its tutorial with a file called `999_temperance.luau`, a short design essay by the author. Its most important claim is worth quoting: **"A hook is not a replacement for systems. They are for enforcing invariants when data changes during different lifecycles."**

When gameplay logic that should run predictably every frame is scattered across hooks, behavior becomes *implicit*: it is triggered indirectly through a cascade of changes, split across many small callbacks that fire in surprising order. That is harder to reason about, harder to debug, and harder to optimize than a system you can read top to bottom.

## The same rule, twice

**Gameplay hiding in a hook**

```luau
world:set(Health, jecs.OnChange, function(entity, id, value)
	if value <= 0 then
		world:add(entity, Dead)      -- a decision
		drop_loot(entity)            -- two side effects
		award_xp(world:get(entity, KilledBy))
	end
end)
```

When does this run? Whenever anything, anywhere, sets `Health`. Half-way through a damage loop, inside another hook, during replication.

**The same logic as a system**

```luau
local function death(world: jecs.World)
	local dying = world:query(Health):without(Dead):cached()
	return function()
		for entity, health in dying do
			if health <= 0 then
				world:add(entity, Dead)
			end
		end
	end
end
-- loot and xp are their own systems, reading the Dead tag after this one
```

It runs at one line in the startup list, after damage and before cleanup, and you can read it top to bottom.

| Belongs in a hook / signal | Belongs in a system |
| --- | --- |
| Destroying the Instance held by a component | Moving entities every frame |
| Stamping an attribute or registering with a debugger | Applying damage, checking deaths, awarding score |
| Recording a change for replication | Deciding *what* to replicate and when |
| Validating that a value is well-formed | AI decisions, spawning waves, cooldown timers |

The same essay makes two more points that shape good JECS code:

- **Not every distinction needs its own component.** "Every semantic distinction deserves its own component or tag" is *sometimes* right, but often a field, a bitset or a local data structure represents the problem better. Match the representation to the problem.
- **Relationships are powerful but every pair is a unique component.** Overusing them silently multiplies archetypes and churn. Use pairs where you actually need entity-to-entity links and queries over them.

> **A practical rule of thumb**
>
> If you could not explain when a piece of code runs by pointing at your system list, it should probably not be in a hook. Keep hooks tiny, side-effect-only, and free of gameplay decisions.

**Key points**

- Hooks enforce invariants at the moment data changes; systems own gameplay logic.
- Logic scattered across callbacks becomes implicit and order-dependent.
- Not every distinction needs a component; not every link needs a pair.
