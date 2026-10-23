# Interfaces and types

*Name a shape without a class: `interface`, array types, `Partial` and spread, and the checks only the type checker makes.*

Alloy · Types, traits and extensions · 10 min

---

Your shop passes item data around as plain tables: `{ id = 'sword', name = 'Sword', count = 1 }`. It comes from a DataStore, so a struct, with its metatable and `new`, is more than it needs. You only want to name the shape, so the editor catches `item.cout` before a player does.

Luau's `type` does that, and Alloy passes it through unchanged. Alloy adds `interface`, a shape with a body.

## Interfaces name a shape

**items.aly**

```alloy
interface Named
  name: string
end

interface Item extends Named
  read id: string
  count: number
  tags: read string[]
end
```

`extends` adds a base shape. `read id` is a field other code may read but not write, and `read string[]` is an array the receiver may not change. A field takes `read` or `write`, never `private`: other code sees an interface whole.

**The ship artifact**

```luau
local __alloy = require("@alloy") type Named = { name: string }

type Item = Named & { read id: string, count: number, tags: __alloy.ReadArray<string> }

 return nil
```

Each interface folds onto its header line as a Luau `type`, and the lines after it go blank, so line 5 is still line 5. The base joins with `&`. No table and no metatable: nothing here runs. Line 1 loads the runtime only because `ReadArray` lives there.

Alloy spells array types the short way. `T[]` is `Array<T>`, the std array with its methods (`map`, `filter`, `reduce` and the rest). `read T[]` is `ReadArray<T>`, the readers alone, and `write T[]` is `WriteArray<T>`, which has only `push`.

`Partial<T>` makes every field of a shape optional, and `Readonly<T>` makes every field read-only. A spread, `{ ...item, name = name }`, copies fields into a new table, and a later key wins.

```alloy
import { Readonly } from '@alloy/std/types'

function total(items: read Item[]) -> number
  return items:reduce(function(sum: number, item) return sum + item.count end, 0)
end

function renamed(item: Item, name: string) -> Readonly<Item>
  return { ...item, name = name }
end
```

`Partial` and `Readonly` are imported from `@alloy/std/types`. `read Item[]` tells the caller that `total` never pushes or removes. The spread emits a call to `__alloy.spread`, which builds a new table, so the caller's `item` keeps its old name.

## Checks only the type checker makes

`x satisfies T` checks a literal against `T` and gives it that type. `~T` is any value that is not a `T`; start with `~nil`. `f<<number>>(x)` passes type arguments at a call. A single `<` reads as a comparison (`single_angle_call`).

Kalo's sandbox runs the ship artifact and has no type checker. You see these reports in your editor and under `alloy flux`, not here.

```alloy
type Settings = { volume: number, music: boolean }

const defaults = { volume = 0.5 } satisfies Settings -- reported: music is missing
const label: ~nil = 'ready' -- any value but nil

function first<T>(xs: { T }) -> T?
  return xs[1]
end
const top = first<<number>>({ 3, 1, 2 })
```

In the ship, `satisfies` becomes a cast, `({ volume = 0.5 } :: Settings)`, and `<<number>>` disappears. The check artifact keeps both for the type checker.

**Key points**

- `type` passes through to Luau; `interface … extends …` is a shape with a body. Neither leaves anything at run time.
- `T[]`, `read T[]` and `write T[]` are `Array`, `ReadArray` and `WriteArray`; `Partial<T>` and `Readonly<T>`, from `@alloy/std/types`, reshape a type.
- `{ ...a, ...b }` builds a new table, and a later key wins.
- `satisfies` reports a missing key but not an extra one. It, `~nil` and `f<<T>>(x)` are checked by the type checker, which the sandbox does not run.
