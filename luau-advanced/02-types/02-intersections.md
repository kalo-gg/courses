# Intersections

*A & B is a value that is both shapes at once, and an intersection of arrows is how Luau spells an overload.*

Luau Advanced · The Type System, Deeply · 10 min

---

A union `A | B` is a value that is *one of* two things, and using it means proving which. An **intersection** `A & B` is the mirror: a value that is *both at once*, so every member's guarantees hold with no proof required.

That sounds exotic until you notice how often the shape appears. Anything described by more than one concern — an entity that is both positioned and named, a handle that is both readable and closable — is an intersection whether or not you write it that way.

**A table that satisfies two shapes**

```luau
--!strict
type Named = { name: string }
type Counted = { count: number }
type Row = Named & Counted

local row: Row = { name = "gold", count = 3 }
print(row.name, row.count)
```

Prints `gold	3`. `row` is a `Named` everywhere a `Named` is wanted and a `Counted` everywhere a `Counted` is, with no narrowing in between.

The second place intersections turn up is **function overloads**. Luau has no `overload` keyword: an intersection of arrow types *is* the overload set. `((number) -> string) & ((string) -> string)` is a function you may call with either, and the checker picks the branch matching the argument.

## Unions and intersections together

|  | Union `A \| B` | Intersection `A & B` |
| --- | --- | --- |
| A value is… | one of the two | both at once |
| To create one | supply either shape | supply everything both shapes demand |
| To use one | narrow it first, or use only what both share | use anything either shape offers |
| Typical use | "a result or an error" | "positioned and named", or a function overload set |

The two are duals: a union is easy to produce and awkward to consume, an intersection the reverse.

**Overloading by intersection**

```luau
--!strict
type Format = ((number) -> string) & ((string) -> string)

local format: Format = (function(v: any): string
	if typeof(v) == "number" then
		return string.format("%.2f", v)
	end
	return v
end) :: any

print(format(1.5), format("raw"))
```

Prints `1.50	raw`. One implementation, two call shapes. The cast is the honest part: the body handles both cases dynamically, and the type is the promise made to callers.

That cast is worth a warning. An intersection of arrows describes a contract the checker will hold *callers* to; it cannot verify that one dynamic body honours both branches. If you write the overload, you own the correspondence between the two.

**Key points**

- `A & B` is a value that is both shapes at once — the dual of a union.
- Creating one is harder than creating a union; using one is easier.
- An intersection of arrow types is how Luau spells a function overload set.
- The checker holds callers to that contract but cannot verify one dynamic body against both branches.
