# Cleanup traits and Exclusive

*Decide what happens to linked entities when a component, relationship or target is deleted.*

JECS · Relationships · 11 min

---

When an entity that is *used as* a component, tag, relationship or target gets deleted, JECS must make sure nothing dangles. **Cleanup traits** let you choose the policy per id. A trait is a pair of a *condition* and an *action* added to the id itself.

|  | Meaning |
| --- | --- |
| **Condition** `jecs.OnDelete` | the id itself (component, tag or relationship) is deleted |
| **Condition** `jecs.OnDeleteTarget` | an entity used as the *target* of this relationship is deleted |
| **Action** `jecs.Remove` | remove the id / pair from every entity that has it (default) |
| **Action** `jecs.Delete` | delete every entity that has the id / pair (cascading) |

**The four combinations**

```luau
local pair = jecs.pair

-- (OnDelete, Remove): default. Deleting the Archer tag removes it from all archers.
world:add(Archer, pair(jecs.OnDelete, jecs.Remove))

-- (OnDelete, Delete): deleting the tag deletes every entity that has it. Dangerous.
world:add(Projectile, pair(jecs.OnDelete, jecs.Delete))

-- (OnDeleteTarget, Remove): deleting a player removes (OwnedBy, player) from their loot.
world:add(OwnedBy, pair(jecs.OnDeleteTarget, jecs.Remove))

-- (OnDeleteTarget, Delete): deleting a parent deletes its children. ChildOf uses this.
world:add(Attached, pair(jecs.OnDeleteTarget, jecs.Delete))
```

Read a trait as a sentence: *when the condition happens, do the action*. `pair(jecs.OnDelete, jecs.Delete)` on `Enemy` means "when the `Enemy` id itself is deleted, delete every entity that has it"; `pair(jecs.OnDeleteTarget, jecs.Remove)` on `OwnedBy` means "when an owner is deleted, strip the pair from the things it owned and leave them alive". The default is `(OnDelete, Remove)`, which is why an unconfigured relationship already behaves safely — and why writing it out is documentation rather than a change.

> **Declare traits before first use**
>
> Traits (and hooks) are read when JECS first creates storage for an id, which happens the first time it is added to any entity. Add cleanup traits right after creating the component or relationship, before you use it anywhere.

## Exclusive relationships

By default an entity can hold the same relationship with many targets (`Likes` apples *and* oranges). Adding `jecs.Exclusive` to a relationship limits it to **one target at a time**: adding a new target replaces the old one in place, and the old pair's removal hook runs. `ChildOf` is exclusive out of the box.

```luau
local Targeting = world:entity()
world:add(Targeting, jecs.Exclusive)

world:add(turret, pair(Targeting, enemy1))
world:add(turret, pair(Targeting, enemy2)) -- replaces enemy1

print(world:target(turret, Targeting, 0) == enemy2) -- true
print(world:target(turret, Targeting, 1))           -- nil
```

**Key points**

- Cleanup traits are `pair(Condition, Action)` on an id: `OnDelete`/`OnDeleteTarget` × `Remove`/`Delete`.
- The default is `(OnDelete, Remove)`; `Delete` actions cascade.
- `jecs.Exclusive` restricts a relationship to one target; `ChildOf` is exclusive and cascades deletes.
- Configure traits immediately after creating the id, before it is used.
