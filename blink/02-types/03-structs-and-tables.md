# Structs, maps and sets

*One fixed shape, one keyed table, one set of flags, and only one of them is free.*

Blink · The Type System · 10 min

---

Three declarations produce a table on the Luau side, and they are not interchangeable. A `struct` is a fixed shape: the keys are known when the file is compiled. A `map` is a keyed table whose keys are not known until you write one. A `set` is a closed list of names, each of which is on or off, which is what you want for flags.

**types.blink**

```blink
struct Loadout {
    Weapon: string(1..24),
    Ammo: u8(0..60)
}

map Scores = { [f64]: u16 }
set Perks = { Sprint, Reload, Scope }
```

| Blink | The Luau type it generates |
| --- | --- |
| `struct Loadout { … }` | `{ Weapon: string, Ammo: number }` |
| `map Scores = { [f64]: u16 }` | `{[number]: number}` |
| `set Perks = { Sprint, … }` | `{Sprint: boolean,Reload: boolean,Scope: boolean,}` |

Copied from a real `Types.luau`, spacing and all. Note the set's `boolean`: the docs describe a set as a map of static keys to `true`, and the generated type says `boolean`.

## Merging and nesting

**types.blink**

```blink
struct Base { a: u8 }
struct Merged { ..Base, b: u8 }
struct Nested { inner: struct { x: u8 } }
```

`Merged` is `{ a: number, b: number }` in Luau: a table union, flattened at compile time.

The `..Base` form copies another struct's fields into this one. It happens while the file is compiled, so `Merged` is one table on the wire and one Luau type, not two nested ones. Nesting is the other option and is written out: a field whose type is an anonymous `struct { … }` really is a table inside a table.

That difference is what decides between a struct and a map. A struct's field names are compiled away: the reader knows the order, so only the values are sent. A map has to send every key beside every value, because nothing on the receiving end knows what the keys will be. A fixed shape declared as a map pays for that ignorance on every message.

## Your turn

**Key points**

- `struct` for known keys, `map` for keys you learn at run time, `set` for flags.
- A struct's field names are compiled away; a map sends every key with every value.
- `..Other` merges a struct's fields at compile time into one flat table.
- A set becomes `{Name: boolean, …}` in Luau, whatever the docs say about `true`.
