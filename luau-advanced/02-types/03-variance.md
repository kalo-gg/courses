# What is assignable to what

*Mutable containers are invariant, read-only ones are not, and function parameters go the other way.*

Luau Advanced · The Type System, Deeply · 9 min

---

A `Dog` is an `Animal`. Is a `{ Dog }` a `{ Animal }`? The intuitive answer is yes and the correct answer is no, and the reason is not pedantry: it is that a list is something you can **write to**.

**The assignment the checker refuses**

```luau
--!strict
type Animal = { legs: number }
type Dog = { legs: number, breed: string }

local dogs: { Dog } = { { legs = 4, breed = "corgi" } }

-- local animals: { Animal } = dogs
--   ^ refused. If it were allowed, someone holding `animals` could append a
--     plain { legs = 2 } — and `dogs` would then hold a Dog with no breed.

print(#dogs, dogs[1].breed)
```

Prints `1	corgi`. The commented line is the one the checker rejects; everything else runs.

A mutable container is therefore **invariant**: `{ Dog }` and `{ Animal }` are unrelated, in both directions. That rule is entirely about the writing. Take the writing away and it disappears — which is what the `read` modifier is for. `{ read legs: number }` is a shape nobody can write through, so widening it is safe.

**read makes it safe to widen**

```luau
--!strict
type Animal = { read legs: number }
type Dog = { read legs: number, read breed: string }

local dogs: { Dog } = { { legs = 4, breed = "corgi" } }
local animals: { Animal } = dogs -- allowed: nothing can write through it

print(#animals, animals[1].legs)
```

Prints `1	4`. Same values, same run-time behaviour — the only change is that the type promises no writes, which is enough to make the assignment sound.

## Functions flip

Parameters go the other way round, and this is the part everyone has to think through once.

A function that accepts *any* `Animal` can stand in wherever a function taking a `Dog` is wanted: whatever dog arrives, it copes. The reverse fails — a function that insists on a `Dog` cannot be handed to code that will call it with an arbitrary `Animal`. Parameters accept **wider** types than required; return values may be **narrower**.

**Key points**

- A mutable container is invariant: `{ Dog }` and `{ Animal }` are unrelated in both directions.
- Invariance exists to protect writers, so a `read`-only container is safe to widen.
- Function parameters accept wider types than required; return values may be narrower.
- `read` and `write` are type-level modifiers with no run-time effect — `table.freeze` is the run-time tool.
