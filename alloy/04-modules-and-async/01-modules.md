# Modules and imports

*`export` on what leaves a file and `import` on what arrives, with services, the std and namespaces.*

Alloy · Modules, Results and async · 11 min

---

A Luau module ends with `return { … }`, a list you keep in step with the functions above it by hand. Every file that uses it opens with `local add = require(path).add` lines. Rename one function and both lists go stale.

Alloy writes both ends as declarations. `export` marks what leaves a file and `import` names what arrives. The compiler writes the `require` calls and the return table.

## Export and import

**inventory.aly**

```alloy
export type Item = { id: string, count: number }

export const MAX_STACK = 99

export function stack(a: Item, b: Item) -> Item
  return { id = a.id, count = math.min(a.count + b.count, MAX_STACK) }
end

export default { name = 'Backpack', slots = 12 }
```

`export` goes in front of any declaration: `const`, `function`, `struct`, `enum`, `type` and the rest. The build gathers the values into one `return { MAX_STACK = …, stack = …, default = … }` on the last line. `export default` adds one unnamed value beside the named ones. A top-level `function` emits as `local function`, because Alloy has no globals.

**main.aly**

```alloy
import bag, { stack, MAX_STACK as CAP, type Item } from './inventory'
import * as Inventory from './inventory'
import type { Profile } from './profile'
import { Players } from '@game'
import { HashMap } from '@alloy/std/collections'

const owners: HashMap<Player, Profile> = new HashMap()
const sword: Item = { id = 'sword', count = 1 }
print(bag.name, Inventory.MAX_STACK == CAP, stack(sword, sword), Players, owners)
```

**main.luau (ship)**

```luau
local __alloy = require("@alloy") local _m1 = require('./inventory') local bag = _m1.default local stack, CAP = _m1.stack, _m1.MAX_STACK type Item = _m1.Item
local Inventory = require('./inventory')

local Players = game:GetService("Players")

const owners: __alloy.HashMap<Player, Profile> = __alloy.HashMap.new()
const sword: Item = { id = 'sword', count = 1 }
print(bag.name, Inventory.MAX_STACK == CAP, stack(sword, sword), Players, owners) return nil
```

Nine lines in, nine out. A bare name reads `.default`, the braces become locals, and `* as M` is the whole table. The `import type` line is blank. The std import writes nothing either: the emit reaches std names through `__alloy`, which is required on line 1.

Two import forms cost nothing at run time. A line that brings in only types emits no `require`, so two modules may import each other's types. And the std's names reach the emit through the runtime.

A Roblox service is an import too: `import { Players } from '@game'` lowers to `game:GetService("Players")` on its own line.

| Module | Names |
| --- | --- |
| `@alloy/std/collections` | `HashMap`, `Set`, `BitSet`, `Queue`, `Heap`, `Symbol` |
| `@alloy/std/async` | `Scope` (and `Future`) |
| `@alloy/std/signal` | `Signal` |
| `@alloy/std/iter` | `Iter` |
| `@alloy/std/serde` | `Serialize`, `Deserialize` |
| `@alloy/std/types` | `Partial`, `Readonly`, `Sink` |
| `@alloy/std/roblox` | `Attributes`, `R15Character`, `R6Character` |

`Result`, `Ok`, `Err`, `Future` and `Array` belong to the language and never need an import. In a project, `[std] globals = "none"` is the default, so a std name used without its import is an `ImportError`. The browser sandbox treats every std name as ambient, so exercises check the import line as text.

> **An import copies the value**
>
> The export table is built when the module loads, and an importer copies each value out of it. After `export local count = 0`, a function that runs `count += 1` later changes only the exporter's local, and the `stale_export` lint reports the write. Export a function that returns the value, or a table whose fields you write.

## Namespaces

Sometimes a group of functions belongs together but does not need a file of its own. `namespace Shop … end` puts them under one name, and outside the group a member reads as `Shop.price`. Members are public by default. A `private` member is reachable only inside the group, and reading it from outside is a compile error.

**shop.aly**

```alloy
namespace Shop
  const TAX = 0.1

  private function round(n: number) -> number
    return math.floor(n + 0.5)
  end

  function price(base: number) -> number
    return round(base * (1 + TAX))
  end
end

print(Shop.price(40)) -- 44
```

The emit is `local Shop = {}`, then each member as a local named `Shop_price`, assigned onto the table on the same line. `Shop_round` is never assigned, so `Shop.round` outside is ``AlloyError: `round` is private to `Shop` ``. `export namespace Shop` sends the whole group from the file.

> **Recognising older code**
>
> `local m = require('./m')` still compiles, and `raw_require` asks for `import m from './m'`. The service paths `'game:Players'` and `'game'` still read in v0.1.0 with a `game_alias` warning, and this release is the last that reads them. `global local x = 1` is removed: it is an `ImportError` that names the `export local` and the `import` line to write instead.

**Key points**

- `export` on any declaration, and the build writes one return table on the last line. `export default` adds one unnamed value.
- `import { a, b as c }` picks names, `* as M` takes the table, and a bare name takes the default.
- Type-only imports and std imports write no `require`. `'@game'` imports lower to `game:GetService`.
- In a project, every std name needs its import, from `@alloy/std/collections`, `/async`, `/signal` and the rest.
- `namespace` groups code under one name. `private` members stay inside it.
