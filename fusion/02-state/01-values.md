# Values

*scope:Value(x) holds state; :set writes it, peek reads it, and a write only fires when the value changed.*

Fusion · State Objects · 9 min

---

A `Value` is the only piece of Fusion state you write to directly. Everything else in this module is derived from one, or reacts to one.

**Create, read, write**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek

local coins = scope:Value(10)

print(peek(coins)) --> 10

coins:set(25)
print(peek(coins)) --> 25
```

`:set` is a method on the value; `peek` is a free function that reads any state object.

> **Why `peek` and not `coins:get()`**
>
> 0.2 had `:get()`, and it was removed. Reading state is two different acts in 0.3: reading *and subscribing* (`use`, inside a `Computed`), or reading *without* subscribing (`peek`). One function that did both was the source of a lot of accidental dependencies, so the two were split apart.

## Writing the same value twice

`:set` compares before it stores. Setting a value to what it already holds does not wake anything up, which is why you can call `:set` in a loop without worrying about redundant work. The comparison runs on every write, so a value fed from a fast stream of events costs nothing extra for as long as the incoming number stays the same.

What counts as "the same" is worth knowing, because it is not simply `==`. Fusion calls it **similarity**, and tables are the interesting case: a frozen table or one carrying a metatable compares by `==`, and **every other table is treated as never similar to anything**.

**Similarity**

```luau
local hp = scope:Value(100)
hp:set(100)   -- no-op: numbers compare by ==

local inventory = scope:Value({ "sword" })
table.insert(peek(inventory), "shield")
inventory:set(peek(inventory))
-- this DOES update: a plain table is never similar, even to itself
```

So mutating a plain table in place and re-setting it works. Do the same to a **frozen** table and nothing fires — there is no force flag to fall back on, so replace the table instead.

> **Prefer replacing over mutating anyway**
>
> Relying on "plain tables are never similar" makes your updates depend on a table not being frozen and not having a metatable — a property nothing in your code states. Building a new table is one line and always fires.

**Key points**

- `scope:Value(x)` is the only state you write; `:set` writes, `peek` reads without subscribing.
- `:set` compares first: an equal number or string is a no-op.
- Plain tables always count as changed; frozen tables and tables with metatables compare by `==`.
- There is no way to force a write past that comparison. `Types.luau` types `:set` as `(Value, newValue, force: boolean?) -> S`, but the implementation in `Value.luau` takes `(self, newValue)` and never reads a third argument: the `force` parameter is vestigial, left over from 0.2.
- Build a new table rather than mutating one in place.
