# Attributes and contracts

*The built-in attributes, your own with targets and defaults, reading them at run time, and contracts the compiler enforces.*

Alloy · Compile-time Alloy · 11 min

---

A fireball needs a cooldown, so someone adds `COOLDOWNS = { Fireball = 2.5 }` in another module. A month later the struct is renamed and the table quietly stops matching. Or your framework calls `Start` on every service, and the one service that forgot it fails only when the game runs.

Alloy's **attributes** put that metadata on the declaration itself, where the compiler can check it.

## The built-in attributes

| Attribute | Goes on | Does |
| --- | --- | --- |
| `@derive(Eq, Debug, …)` | struct, enum | Generates methods from the field list |
| `@sealed` | struct | A write to an undeclared key raises |
| `@test` | function, namespace | A test; blanked from the ship (lesson 4) |
| `@cfg(server)` | function, namespace, local, statement | Runs on one side only |
| `@rename`, `@skip`, `@alias` | field | The field's key in derived tables |
| `@unreliable`, `@immediate`, `@ratelimit`, `@timeout`, `@validate` | remote | How a remote sends and what it accepts |
| `@u8` … `@f32` | parameter, field | The number's width on the wire |
| `@allow(lint)` | a declaration or statement | Quiets that lint there |
| `@native`, `@checked`, `@inline`, `@noinline`, `@deprecated` | function | Luau's own, written to the emit |

Luau's own attributes pass through, and so does Luau's list form `@[native, checked]`, so a Luau file stays valid Alloy.

## Declare your own

**cooldown.aly**

```alloy
import { Attributes } from '@alloy/std/roblox'

attribute cooldown(seconds: number = 1) on struct

@cooldown(2.5)
struct Fireball
  damage: number
end

@cooldown
struct Punch
  damage: number
end

print(Attributes.get(Fireball, cooldown)) -- 2.5
print(Attributes.get(Punch, cooldown)) -- 1
```

The second argument of `Attributes.get` is the declared name, not a string. The declaration emits `local cooldown = __alloy.attribute("cooldown", { "struct" }, { "seconds" })` on its own line.

The targets after `on` say where an attribute may go: `struct`, `function`, `field`, `variant`, `impl` and more. Put it anywhere else and the compile stops with `AttributeError`. A parameter may carry a default, so a bare `@cooldown` reads `1`. At run time, `Attributes.get` reads one parameter as its value, several as a table, and none as `true`.

**A contract**

```alloy
attribute service on impl as
  requires public function Start(self)
end

struct Shop
  open: boolean
end

@service
impl Shop
  function stop(self) -> () end
end
```

This does not compile: **AttributeContract**: "@service requires a public function Start(self); Shop declares none". A clause fixes the member's name, so `naming_convention` leaves `Start` alone.

An `as … end` body turns the attribute into a **contract**. Each `requires` clause names a function or field the target must carry, and the compiler checks it where the attribute is used. A contract emits nothing. It is structural, like an interface, and a struct and its `impl` blocks count as one type.

> **One clause per list entry**
>
> `requires public function each lifecycles(self)` writes one clause for each entry of a list argument. With it, `@provider([ Lifecycle.Init, Lifecycle.Start ])` asks for `Init` and `Start`, and `@provider([ Lifecycle.Start ])` asks only for `Start`.

> **@allow is checked too**
>
> `@allow(too_many_arguments)` quiets one lint over the declaration it sits on, and it moves with the code. A name that no lint has is an `AttributeError`, so a typo cannot quietly silence nothing.

**Key points**

- `attribute name(params) on targets` declares checked metadata; a misplaced use is an `AttributeError`.
- Parameters take defaults, and `Attributes.get(Target, name)` reads the value at run time.
- An `as … end` body of `requires` clauses is a contract, checked where it is used, reported as `AttributeContract`, and emitting nothing.
- Luau's own `@native`, `@checked`, `@inline`, `@noinline` and `@deprecated` pass through to the emit.
