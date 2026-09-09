# Designing components

*Small, data-only components; tags vs fields; singletons; naming; and when a relationship is worth it.*

JECS · Best Practices & Capstone · 10 min

---

Good JECS code starts with good component boundaries. The JECS author's advice is to treat the world like a **database**: normalize data into small components so queries ("joins") stay simple and systems stay focused. A `Renderable` component that any system can position is more useful than a `ZombieData` blob that only the zombie system understands.

| Smell | Better |
| --- | --- |
| One giant `Stats` table with health, speed, damage, team, model… | `Health`, `Speed`, `Damage`, `Team`, `Model` as separate components so each system queries only what it needs |
| A `type = "zombie"` string checked with `if` inside systems | A `Zombie` tag (or a `Kind` component) so systems *query* for it |
| A component holding *behavior* (functions) | Data only; put the behavior in a system |
| A tag toggled every frame (`OnGround` add/remove) | A boolean/enum *field* on a data component; tags for rarely-changing states |
| `targetId` stored as a number in a table | A relationship pair (`pair(Targeting, target)`) when you need to query it; a field when you never do |

**A god component**

```luau
local Stats = jecs.component() :: jecs.Id<{ health: number, speed: number, team: string, model: Model }>

for entity, stats in world:query(Stats) do
	move(entity, stats.speed) -- loads health, team and the model too
end
```

**Normalized**

```luau
local Health = jecs.component() :: jecs.Id<number>
local Speed = jecs.component() :: jecs.Id<number>
local Team = jecs.component() :: jecs.Id<string>
local Model = jecs.component() :: jecs.Id<Model>

for entity, speed in world:query(Speed) do
	move(entity, speed)
end
```

Each system now names exactly what it reads and writes, and Jabby shows four columns instead of one blob.

The two blocks store the same facts and cost the same memory. What changed is what a *query* can say. With `Stats`, every system that touches an entity's speed also loads its team and its model, and a change to any field looks the same to a listener. With four components, a system's query is its interface — you can read what it reads from its first line — and `world:changed(Speed, …)` fires only for speed.

## Guidelines that survive contact with real games

- **Type every component.** `world:component() :: jecs.Id<T>` is cheap and turns a whole class of bugs into editor squiggles.
- **Name every component.** `jecs.meta(C, jecs.Name, "C")` (or `world:set(C, jecs.Name, …)`) makes Jabby, logs and replication readable.
- **Preregister in one shared module.** `jecs.component()`/`jecs.tag()` before `jecs.world()`; server and client then agree on ids for free.
- **Singletons for global state.** `world:set(MatchState, MatchState, …)` beats module-level globals: it is inspectable and replicable.

> **The author's counterweight**
>
> The JECS tutorial closes with a warning against over-splitting: "every semantic distinction deserves its own component" is *sometimes* right, but a field, bitset or local data structure can fit the problem better. Split when it makes a query clearer or a system simpler, not on principle.

**Key points**

- Normalize: small, typed, named, data-only components.
- Tags for rare state changes, fields for per-frame flags, pairs for links you query.
- Split for clearer queries, not on principle.
