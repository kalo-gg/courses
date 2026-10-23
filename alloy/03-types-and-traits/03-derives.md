# Derives and sealed structs

*Let `@derive` write equality, ordering, cloning and saving from the field list, and let `@sealed` catch typos.*

Alloy · Types, traits and extensions · 11 min

---

A player picks a loadout, and you check it against the saved one: `if picked == saved then`. It is never equal, even when every field matches. Two struct values are two tables, and `==` on tables compares identity.

Writing `__eq`, `clone` and `__lt` by hand for every struct is the kind of boilerplate Alloy exists to remove. `@derive` on the line above a struct writes them from its field list.

## Derives write the methods

```alloy
@derive(Eq, Clone)
struct Loadout
  weapon: string
  perks: string[]
end

const saved = new Loadout { weapon = 'bow', perks = [ 'swift' ] }
const picked = saved:clone()
print(picked == saved, rawequal(picked, saved)) -- true  false
```

`Eq` writes `__eq`, which compares field by field, and an array field by its contents. `Clone` writes `clone()`: a new table with the same metatable and its own copy of `perks`.

Each derive writes a fixed set of methods, on the struct's own emitted lines. `Serialize` and `Deserialize` come from `@alloy/std/serde`, along with field attributes that shape the saved table: `@rename`, `@skip`, `@alias`, `@rename_all` and `@deny_unknown_fields`.

| Derive | Writes | Worth knowing |
| --- | --- | --- |
| `Eq` (or `PartialEq`) | `__eq` and `eq` | Table, array, map and set fields compare by contents |
| `Ord` | `__lt`, `__le`, `lt`, `le` | Fields compare in declaration order, and the first difference decides, so `table.sort` works |
| `Debug` | `debug` and `__tostring` | The name and fields, as text |
| `Clone` | `clone()` | Copies collections; shares anything else, an Instance included |
| `Default` | `Name.default()` | Declared defaults kept; the rest at their type's zero (`0`, `''`, `false`, `Vector3.zero`…) |
| `Serialize` / `Deserialize` | `to_table()`, `serialize()` / `Name.from_table(t)` | A missing key keeps its default, so an old save still loads |

> **The lint that finds the bug**
>
> Without `Eq`, `x == new Loadout { … }` is always false, because a table built in the comparison equals no other. The `identity_compare` lint warns about exactly that and suggests `@derive(Eq)`.

```alloy
import { Serialize, Deserialize, skip } from '@alloy/std/serde'

@derive(Serialize, Deserialize)
struct Save
  coins: number
  level: number = 1
  @skip
  cache: string?
end

const data = new Save { coins = 10, cache = 'x' }:to_table() -- { coins = 10, level = 1 }
print(Save.from_table({ coins = 3 })) -- Save { coins = 3, level = 1, cache = nil }
```

`to_table()` writes plain data for a DataStore or a remote, without the `@skip` field. `from_table` puts back the metatable and the defaults, so a save written before `level` existed still loads.

## @sealed catches typos

A struct is open. `settings.volme = 0` creates a new key, and the typo goes unnoticed until someone wonders why the volume never changed. `@sealed` adds a `__newindex` that raises on any key the struct does not declare. Declared fields write as before.

It stops typos, not writes. To stop writes to a declared field, mark the field `read`; the type checker enforces that, not the running game.

```alloy
@sealed
struct Settings
  volume: number = 1
  read music: boolean = true
end

local settings = new Settings {}
settings.volume = 0.5 -- fine: declared
settings.volme = 0 -- error: Settings has no field volme
```

The game raises the error at run time, and the check artifact rejects the write too, so the editor flags it first. `@sealed` puts a metamethod on absent keys, so keep it off structs in a hot loop.

**Key points**

- `==` on two structs compares identity unless the struct derives `Eq`; `identity_compare` warns about it.
- `@derive` writes `Eq`, `Ord`, `Debug`, `Clone`, `Default`, `Serialize` and `Deserialize` methods from the field list.
- `Clone` copies collections and shares everything else, an Instance included.
- `@sealed` raises at run time on an undeclared key; `read` stops writes to a declared field, through the type checker.
