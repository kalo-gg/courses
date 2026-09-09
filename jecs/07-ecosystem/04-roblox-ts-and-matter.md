# roblox-ts and coming from Matter

*Using JECS from TypeScript with @rbxts/jecs and its companions, plus this course's own Matter to JECS mapping.*

JECS · The JECS Ecosystem · 12 min

---

JECS is published to npm as **`@rbxts/jecs@0.11.0`** with hand-written typings. Nearly everything maps one-to-one onto the Luau API, with a few TypeScript-flavoured differences: generics replace `:: jecs.Id<T>` casts, multi-value returns become `LuaTuple`s, and the world comes from a **function**, not a constructor.

**src/shared/world.ts**

```typescript
import { world, OnAdd, Name } from "@rbxts/jecs";

const w = world(); // world(true) for DEBUG mode; the class constructor is private

const Position = w.component<Vector3>();
const Velocity = w.component<Vector3>();
const Health = w.component<number>();

w.set(Position, Name, "Position");

const e = w.entity();
w.set(e, Position, new Vector3(0, 5, 0));
w.set(e, Velocity, Vector3.zero);
w.set(e, Health, 100);

const [pos, hp] = w.get(e, Position, Health); // two or more: LuaTuple<[Vector3?, number?]>

for (const [id, position, velocity] of w.query(Position, Velocity)) {
	w.set(id, Position, position.add(velocity.mul(dt)));
}

w.set(Position, OnAdd, (entity, id, value) => {
	print("Position added to", entity, value);
});
```

| Luau | TypeScript |
| --- | --- |
| `jecs.world()` | `world()` (a `new World()` call does not compile) |
| `world:component() :: jecs.Id<T>` | `w.component<T>()` |
| `local a = world:get(e, A)` | `const a = w.get(e, A)`: one component is **not** a LuaTuple |
| `local a, b = world:get(e, A, B)` | `const [a, b] = w.get(e, A, B)`: two to four are |
| `for e, a, b in world:query(A, B) do` | `for (const [e, a, b] of w.query(A, B))` |
| `world:set(C, jecs.OnAdd, fn)` | `w.set(C, OnAdd, (entity, id, value) => {})` |

Two type-level gotchas are worth meeting here rather than in a red squiggle. `w.get` is piped through `FlattenTuple`, so two to four components come back as a `LuaTuple` you destructure and a single component as a plain `T | undefined` you must not — `const [hp] = w.get(e, Health)` is the mistake, not the fix. And `w.add` is typed for tags only, so `w.add(e, Health)` is a compile error.

## Ecosystem packages on npm

npm mirrors the Wally ecosystem with one trap: `latest` on `@rbxts/planck` is a stale `1.0.0`, so install Planck and its companions from the `rc` dist-tag. `@rbxts/jabby` sits on the 0.10 line and `@rbxts/replecs` moves with a `next` tag; check both before pinning. Versions live in the reference.

## Coming from Matter

| Matter | JECS | Notes |
| --- | --- | --- |
| `Matter.component("Name")` | `jecs.component() :: jecs.Id<T>` + `jecs.meta(C, jecs.Name, "Name")` | Declare before `jecs.world()`. |
| `world:spawn(A(...), B(...))` | `world:entity()` then `world:set` per component | No component constructors; values are plain. |
| `world:insert(id, C(...))` | `world:set(id, C, value)` | `set` is add-or-update. |
| `world:despawn(id)` | `world:delete(id)` | Runs cleanup traits and hooks. |
| `world:query(A, B)` | `world:query(A, B)` | Same loop shape; add `:with`, `:without`, `:cached`. |
| `world:queryChanged(C)` | `world:added/changed/removed(C, fn)` or the `(Previous, C)` pair pattern | Signals and Change tracking. |
| (no equivalent) | Relationships: `jecs.pair`, `ChildOf`, cleanup traits | Relationships. Matter has none. |

**Before: Matter 0.8**

```luau
local Matter = require(ReplicatedStorage.Packages.Matter)

local Health = Matter.component("Health")
local Position = Matter.component("Position")

local world = Matter.World.new()
local zombie = world:spawn(
	Health({ value = 100 }),
	Position({ x = 0, y = 0 })
)

world:insert(zombie, Health({ value = 50 }))

for id, health, position in world:query(Health, Position) do
	print(id, health.value, position.x)
end

world:despawn(zombie)
```

**After: JECS 0.11**

```luau
local jecs = require(ReplicatedStorage.Packages.jecs)

local Health = jecs.component() :: jecs.Id<number>
local Position = jecs.component() :: jecs.Id<Vector3>
jecs.meta(Health, jecs.Name, "Health")
jecs.meta(Position, jecs.Name, "Position")

local world = jecs.world()
local zombie = world:entity()
world:set(zombie, Health, 100)
world:set(zombie, Position, Vector3.zero)

world:set(zombie, Health, 50)

for id, health, position in world:query(Health, Position) do
	print(id, health, position.X)
end

world:delete(zombie)
```

Notice that `Health` is just a number now. Matter wrapped every value in a component instance; JECS stores whatever you give it, so a scalar stays a scalar.

This mapping is this course's, not an official one; Matter's last release is v0.8.5 (December 2024). Three habits to unlearn:

- **Immutability.** In JECS, `set` is the signal: write through `world:set` when hooks or signals must fire, mutate in place when they must not.
- **Hooks inside systems.** `useEvent` and `useThrottle` become Planck run conditions on the system.
- **Links as ids in data.** Where Matter stored `{ owner = id }`, JECS offers a pair with cleanup traits.

**Key points**

- `@rbxts/jecs@0.11.0`: `world()` not `new World()`, `w.component<T>()`, destructure `w.get` only when reading two or more components, `w.add` for tags only, hooks via `w.set(C, OnAdd, cb)`.
- Companions: `@rbxts/planck@rc` (ignore `latest`), `@rbxts/replecs@0.3.1`, `@rbxts/jabby@0.5.1` (JECS 0.10 only); `rbxts-transformer-jecs` inlines query loops at compile time.
- Matter → JECS is our own mapping: `component("Name")` → `jecs.component()` + `jecs.meta`; `spawn`/`insert`/`despawn` → `entity`+`set`/`set`/`delete`; `queryChanged` → signals or `Previous` pairs; `Loop` hooks → Planck.
- Matter has no relationships; in JECS, entity-to-entity links are pairs with cleanup traits.
