# Your first Alloy script

*Build parts with `new` and an initializer, name things with `const`, and meet arrays, destructuring, `in` and `is`.*

Alloy · Foundations · 11 min

---

A spawn platform takes six lines of Luau: `Instance.new`, four property writes, and a `Parent` that has to come last so the part enters the world with its properties already set. Alloy writes the same part as one expression, and puts `Parent` last for you.

## Construct with new

**platform.aly**

```alloy
const platform = new Instance('Part') {
  Parent = workspace,
  Anchored = true,
  Size = new Vector3(8, 1, 8),
  Position = new Vector3(0, 10, 0),
}
```

`new Instance('Part')` constructs, and the braces after it are an **initializer**: one field assignment each.

**What the compiler emits**

```luau
local platform = Instance.new('Part')
  local _parent1 = workspace
  platform.Anchored = true
  platform.Size = Vector3.new(8, 1, 8)
  platform.Position = Vector3.new(0, 10, 0)
platform.Parent = _parent1 return nil
```

`Parent` was written first and is still assigned last: the emit keeps it in `_parent1` and assigns it on the closing brace's line. `new Vector3(…)` is `Vector3.new(…)`.

`const` is a binding that is never reassigned, and a reassignment is `ConstError` at compile time. It passes through as Luau's own `const`, which Luau Fundamentals writes `local const`, except when the initializer needs statements after the binding, as above: there it emits `local`, and Alloy still refuses the reassignment. Write `const` wherever a name never changes and nothing writes into its value, as the `prefer_const` lint asks.

`c ? a : b` is the ternary, which emits `(if c then a else b)`. Leave a space before its `:`, because `a:b` is a method call.

## Arrays and patterns

**loot.aly**

```alloy
const scores = [ 3, 8, 5, 10 ]
print(scores:map(function(s) return s * 2 end):join(', ')) -- 6, 16, 10, 20

const [ first, ...rest ] = [ 'sword', 'shield', 'bow' ]
const { name, hp = health } = { name = 'Ada', hp = 80 }
print(first, #rest, name, health) -- sword  2  Ada  80

const hp = health > 50 ? 'healthy' : 'hurt'
print(`{name} is {hp}`, 'bow' in rest) -- Ada is healthy  true

const part = new Instance('Part')
print(part is Part, part is Model) -- true  false
```

`[ ]` makes an array with the std's methods, such as `map`, `filter` and `join`, so this file gains the runtime `require` on line 1. `x is Part` emits a `typeof` and `IsA` test.

An array pattern takes items by position, and `...rest` takes the tail as an array. A table pattern takes fields by name, and `field = local` renames: `{ name, hp = health }` binds the locals `name` and `health`.

`x in t` searches an array by value. `2 in { 5, 6 }` is a compile error, though: a `{ }` literal's items sit under the keys 1 and 2, so write `[ 5, 6 ]`.

> **Recognising older code**
>
> `Vector3.new(…)` and `Instance.new('Part', workspace)` still compile. `prefer_new` asks for `new Vector3(…)`, and `instance_new_parent` reports the two-argument form, which parents the part before its properties are set. A `local` that is never reassigned or written into draws `prefer_const`.

**Key points**

- `new Instance('Part')` and `new Vector3(…)` construct; an initializer's braces set one field per line.
- An initializer always assigns `Parent` last, whatever order you write it in.
- `const` cannot be reassigned (`ConstError`); write it wherever a name never changes and its value is not written into.
- `[ ]` is an array with `map`, `filter` and `join`; patterns destructure by position or by field, with `field = local` to rename.
- `x in t` searches an array's values, and `x is Part` tests a type; `in` on a `{ }` literal with items is a compile error.
