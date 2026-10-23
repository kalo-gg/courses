# Traits

*Write a contract once, have every impl checked against it, and let operator traits write the metamethods.*

Alloy · Types, traits and extensions · 11 min

---

Swords, shields and potions all show a tooltip, so each struct writes a `label` method. Nothing says it must. The day someone adds a `Bow` without one, the tooltip breaks on a live server.

A **trait** writes the contract down once: the methods a type must have, with default bodies for the ones it may skip. Each `impl Trait for Struct` is checked against it at compile time.

## A trait is a contract

**items.aly**

```alloy
trait Describe
  function name(self) -> string
  function label(self) -> string
    return `[{self:name()}]`
  end
end

struct Sword
  damage: number
end

impl Describe for Sword
  function name(self) -> string
    return `Sword ({self.damage})`
  end
end

print(new Sword { damage = 12 }:label()) -- [Sword (12)]
```

`Sword` writes `name`, the one method with no body, and gets the trait's `label`, which calls `name` through `self`.

**The same file, with name deleted from the impl**

```text
items.aly:12:6 error(3.6): TraitContract: `impl Describe for Sword` does not write `name`; the trait declares it
```

A missing method stops the build, and so does one with the wrong number of parameters. `alloy doc 3.6` explains the section.

A trait also works as a bound. `<T: Describe>` lets a function take any type that implements it, and `<T: A & B>` asks for two. Bounds go on functions and methods, and a trait itself takes no type parameters.

```alloy
function tooltip<T: Describe>(item: T) -> string
  return `{item:label()} | {item:name()}`
end

print(tooltip(new Sword { damage = 12 })) -- [Sword (12)] | Sword (12)
```

The type checker sees `item` as a `T` with `Describe`'s methods, so both calls type. Passing a struct with no `impl Describe` stops the build with a `BoundError` (3.6).

## Operator traits

Some trait names are built in. Implement one and Alloy sets the matching metamethod after the impl's `end`, on the same line. `Add`'s `add` becomes `__add`, `Lt`'s `lt` becomes `__lt`, and `Display`'s `to_string` becomes `__tostring`, which `print` and interpolation use. `Drop`'s `drop` becomes the `Destroy` that `delete` runs. These names need no import, and `alloy doc Traits` lists them.

```alloy
struct Coins
  amount: number
end

impl Lt for Coins
  function lt(self, other: Coins) -> boolean
    return self.amount < other.amount
  end
end

impl Display for Coins
  function to_string(self) -> string
    return `{self.amount}c`
  end
end

print(new Coins { amount = 5 } < new Coins { amount = 9 }, new Coins { amount = 5 }) -- true  5c
```

Line 9 emits as `end Coins.__lt = Coins.lt`, and line 15 as `end Coins.__tostring = Coins.to_string`. The method stays callable by name too: `a:lt(b)`.

**Key points**

- A trait lists method signatures; a body makes a method a default that impls may skip.
- `impl Trait for S` must write every required method, or the compile stops with `TraitContract` (3.6).
- `<T: Trait>` and `<T: A & B>` bound a function's generic. A trait takes no type parameters.
- Operator traits set a metamethod after the impl: `Add` → `__add`, `Display` → `__tostring`, `Drop` → the `Destroy` that `delete` runs.
