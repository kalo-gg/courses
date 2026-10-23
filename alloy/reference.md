# Alloy reference

*Alloy 0.1.0, released October 23, 2026. Checked on October 23, 2026.*

---

## Safe access and child lookup

Each operator emits an `if … then … else` on a temp, on the line you wrote it.

### `Safe index`

```luau
a?.b
```

Nil when `a` is nil, else `a.b`. One `?` guards the rest of the chain, so `a?.b.c` is nil when `a` is. `a?.b?.c` emits `local _1 = (if a == nil then nil else a.b) local x = (if _1 == nil then nil else _1.c)` on one line.

> The temp runs in front of the statement. Where that would change the result (after `and`, `or` or `??`, in a later branch, in a loop condition) the chain re-reads its path inline or runs in a closure called in place.

### `Safe method call`

```luau
a?:m(x)
```

Nil when `a` is nil, and then `x` is never evaluated. Else `a:m(x)`.

### `Safe index by key`

```luau
a?[k]
```

Nil when `a` is nil, else `a[k]`.

### `Safe call`

```luau
f?(x)
```

Nil when `f` is nil, and `x` is never evaluated. Emits `(if f == nil then nil else f(x))`.

### `Nil coalescing`

```luau
a ?? b
```

`a` when it is not nil, else `b`.

> It tests nil only, so `false ?? true` is `false`. That is the difference from `or`, which drops `false`: `settings?.muted ?? true` keeps a stored `false`.

### `Nil-coalescing assignment`

```luau
a ??= b
```

Assigns `b` only when `a` is nil. On a plain local it emits `x = if x == nil then v else x`; an indexed target emits `if t.a == nil then t.a = v end`, its path evaluated once.

### `Non-nil assertion`

```luau
a!
```

Raises when `a` is nil, with the source text in the message, and drops the `?` from the type. `m.PrimaryPart!` emits `(if m.PrimaryPart == nil then error("m.PrimaryPart is nil") else m.PrimaryPart)`.

> Inside a safe chain, `!` also ends the guard: what follows it is no longer nil-guarded.

### `Child lookup`

```luau
a->b
a->'Name With Spaces'
a->[expr]
```

`a:FindFirstChild("b")`, nil when the child is missing and nil when `a` is, so it needs no `?`. The right side is a name, a string, an interpolated string or `[expr]`.

> A receiver whose type is known to be something other than an `Instance` is an `AlloyError` (4.1); an untyped one is not checked. After a parameter list, `-> T` is a return type instead.

### `Blocking child lookup`

```luau
a=>b
```

`a:WaitForChild("b")`. With `[emit] wait_timeout` set, the timeout is passed, the call can return nil, and `=>` guards the rest of the chain like `->`. Without it the call waits forever and the result is a plain `Instance`.

> `alloy init` writes `wait_timeout = 5`, so a new project gets the nil-able form. The key's own default is unset, and so is the browser sandbox's, where `=>` never returns nil.

## Sugar

### `Ternary`

```luau
c ? a : b
```

Emits `(if c then a else b)`.

> A `:` touching a name is a method (`x:upper()`), so the else marker takes a space before it.

### `new`

```luau
new Vector3(0, 10, 0)
new Name(…)
new Name { field = value }
```

Constructs a value. For a Roblox datatype, an Instance or any class it emits `Thing.new(…)`. For a struct, `new Name(…)` calls the `new` or `New` its impl writes, and `new Name { … }` is the fields form, the only way to build a struct that writes no constructor.

> `prefer_new` reports `Vector3.new(…)` and `alloy flux --fix` rewrites it. `new Thing():m()` chains as `(new Thing()):m()` does.

### `Initializer`

```luau
new Instance('Part') {
  Anchored = true,
  Parent = workspace,
}
```

Braces after a constructor call set fields on the new value, one assignment each, on the same line. `Parent` is always assigned last, whatever order you write it in, so the Instance enters the tree with its properties set.

> Replaces `Instance.new(class, parent)`, which parents first; the `instance_new_parent` lint reports that form.

### `Array literal`

```luau
[ 1, 2, 3 ]
[ [1, 2], [3, 4] ]
```

An `Array<T>` with the std's metatable, so `xs:map(f)` and `xs:filter(f)` work on it. Items separate with `,` or `;`. Emits `__alloy.Array.from({ … })`.

> `[[` opens a Luau long string, so a nested array needs a space: `[ [1, 2] ]`.

### `Destructuring`

```luau
local [head, ...rest] = xs
local { name, hp = health } = player
```

An array pattern takes items by position and `...rest` takes the tail as an array. A table pattern takes fields by name, and `field = local` renames: `{ name, hp = health }` binds locals `name` and `health` from the fields `name` and `hp`. `const` takes both forms.

> A parameter can be a table pattern too, `function draw({ x, y }: Point)`, but it must state its type.

### `Spread`

```luau
{ ...base, x = 1 }
```

Copies `base` into a new table, then the keys after it. Emits `__alloy.spread(base, { x = 1 })`.

> A positional entry after a spread is an error.

### `Membership`

```luau
x in t
```

Searches an Array by value, a `Set` or `HashMap` through `:has`, a string by substring, and a plain table by key. Emits `__alloy.contains(t, x)`.

> `2 in { 5, 6 }` is a compile error, because a `{ }` literal's items sit under keys 1, 2: write `[ 5, 6 ]`. A keyed literal, `kind in { rect = true }`, is fine.

### `Type test`

```luau
x is Part
x is not T
```

A primitive, Roblox datatype, Instance class, `Enum.Name`, or Alloy struct or enum. Emits `type`, `typeof`, `IsA` or a metatable check: `x is Part` is `(typeof(x) == "Instance" and x:IsA("Part"))`. In an `if`, a test on a plain name narrows it.

> A trait, an interface, a remote or an attribute is refused: no value carries one. Replaces `typeof(x) == 'T'` checks (`manual_type_test`).

### `const`

```luau
const MAX_HP = 100
```

A binding that cannot be reassigned, checked by Alloy across scopes: a reassignment is `ConstError` (6.1), reported before Luau sees the file. It passes through to Luau's own `const`, except where the initializer needs statements after the binding (`new Instance('Part') { … }`), which emits a `local` the compiler still guards.

> The freeze is shallow: fields of a `const` table can still be written, and the pedantic `const_mutation` lint reports each write. `prefer_const` asks for `const` on a local that is never reassigned and never written into, so a table you fill or an Instance whose properties you set stays `local` and neither lint fires.

### `Bit operators`

```luau
a band b   a bor b   a bxor b
bnot a     a shl n   a shr n
```

Words for `bit32.band`, `bor`, `bxor`, `bnot`, `lshift` and `rshift`. Operands are 32-bit unsigned.

> `$bnot(x)` takes a parenthesized operand.

### `Method reference`

```luau
local f = obj:method
```

A `:` method name with no call is the method bound to `obj`, a function to pass along. Emits `__alloy.bind(obj, obj.method)`.

## Structs

### `struct`

```luau
struct Vec2
  x: number
  y: number = 0
end
```

A record with fields and optional defaults. Emits a class table with `__index`, a `__call` that fills the defaults and sets the metatable, `Vec2.new`, a table type, and a `__tostring` on the same lines as the source.

> `as` puts members on the header line, `struct Vec2 as x: number end`. A body on the lines below takes no `as`: `redundant_as` reports one there.

### `Construction contract`

```luau
new Vec2 { x = 1 }
```

A field with no default left unset is `ConstructorError: new P { ... } leaves x unset`. An unknown field is `StructError: P has no field z`. Both are compile errors (book 3.6).

> A bare `P { x = 1 }` is a `ConstructorError` too: a struct never constructs without `new`.

### `Default printing`

```luau
print(new Vec2 { x = 1 })   -- Vec2 { x = 1, y = 0 }
```

Every struct prints its name and fields through `__alloy.show_struct`. A `to_string` in `impl Display` replaces it.

### `impl`

```luau
impl Vec2
  function len(self) -> number
    return math.sqrt(self.x ^ 2 + self.y ^ 2)
  end
end
```

Methods for a struct or an enum, emitted as functions on the class table inside a `do … end`. A method written as `new` or `New` is the constructor `new Vec2(…)` calls.

> A function without `self` is a static: call it with `.`. Calling one with `:` shifts every argument by one, and `static_call` reports it.

### `read / write`

```luau
read id: string
write sink: string[]
```

A read-only or write-only field, checked by the type checker. `read T[]` is `ReadArray<T>` and `write T[]` is `WriteArray<T>`.

### `private / public`

```luau
private count: number = 0
private function reset(self) … end
```

A private field or method belongs to the struct's own impl. The word emits nothing: the check artifact leaves the member out of the public type, so outside use is a type error under `alloy flux`, and `private_access` reports it in the same file. `public` is the default.

> A private field with no default must still be set in `new S { }`.

### `class`

```luau
class Name … end
```

The Luau classes RFC syntax. v0.1.0 parses it, reports `StructError: class is parsed and not compiled yet`, and blanks the block.

> Not yet. A `struct` with an `impl` is the form that runs.

## Enums and patterns

### `Unit enum`

```luau
enum Color
  Red
  Green
end
```

A string union at run time: `Color.Red == 'Red'`. Emits the strings on a table, `Color.is(v)`, a `__tostring`, and `type Color = "Red" | "Green"`.

> A unit value is a string, so its methods are called on the enum: `Color.describe(c)`, not `c:describe()`.

### `Payload enum`

```luau
enum Msg
  Chat(string)
  Quit
end
```

A tagged table per payload variant: `Msg.Chat('hi')` is `setmetatable({ tag = "Chat", _1 = 'hi' }, Msg)`. It prints as `Msg.Chat("hi")`, strings quoted. Unit variants in the same enum stay strings and print as their name.

### `Generic enum`

```luau
enum Slot<T>
  Empty
  Full(T)
end
```

Type parameters a payload may name. `impl Slot<T>` names them in a method.

### `match`

```luau
match value as v with
  case Chat(text) then …
  case Quit then …
  default …
end
```

A statement or an expression. `as name` names the value once for every arm. `match a, b with` matches several values. As an expression it emits an `if`-chain inside parentheses; each arm's last line is its value.

> An arm with statements before its value works after `local x =`, `x =` or `return`. `return` in an arm leaves the enclosing function.

### `Patterns`

```luau
Move(x, y)   _   1   'a'   nil   Color.Red
Vec2 { x = 0, y }   [first, ...rest]   A or B
```

A variant with its payload, a binding or `_`, a literal, a dotted path, a struct pattern (`y` alone binds the field), an array pattern, and `or` alternatives.

> A bare capitalized name is a unit variant, so one no enum has is an error, and `case MAX` with a `MAX` local in scope is too: compare it in a guard. A lowercase name always binds a new one; when it shadows a local, `pattern_shadows_local` warns.

### `where guard`

```luau
case n where n > 5 then
```

An arm that matches only when the condition holds.

> A guard proves nothing about coverage. The older `case n and n > 5` still parses; `match_guard_and` rewrites it.

### `Exhaustiveness`

```luau
error(4.2): ExhaustiveMatch
```

A `match` with no `default` must cover every variant: `this match is not exhaustive: Msg has no arm for Quit; add it or a default arm`. A literal, struct or array scrutinee always needs a `default` or a catch-all binding.

> In a single-file compile, an enum imported from another file has no variant list, so its `match` asks for a `default`. A `default` under arms that already cover everything draws `unreachable_default`.

### `default`

```luau
default …
```

The fallback arm. It comes last: a `case` after it is an error.

### `$matches`

```luau
$matches(r, Ok(_))
```

A pattern test without a `match`: `true` when the value fits. Any `case` pattern works, and names in it bind nothing.

## Conditional bindings

### `if local / if const`

```luau
if const character = player.Character then
  …
end
```

Binds and tests in one step; the name exists only in the branch. Emits `do local _c1 = … if _c1 then local character = _c1 … end end`. `elseif local` works the same way.

> Write `if const` when the branch only reads the name (style guide, `prefer_const`), and `if local` when it reassigns the name or writes into its value (`const_mutation`).

### `Pattern binding`

```luau
if const Ok(v) = result then
```

Any `case` pattern binds: the branch runs only when the value fits.

### `where and chains`

```luau
if const p = a where p.ready; const c = p.Character then
```

`where` filters a binding. `;` chains bindings, each seeing the last, and each may carry its own `where`.

### `while const / while local`

```luau
while const job = queue:pop() where job.ready do
```

Loops while the binding is non-nil and the `where` holds; a failed `where` ends the loop, it does not skip. Emits `while true do local _c1 = … if not (_c1 and (…)) then break end …`.

> Write `while const`: `prefer_const` reports a `while local` name that is never reassigned.

### `Guard clause`

```luau
if not const item = find(id) then return end
```

Runs the block when the binding fails, and keeps `item` in scope after it. Emits `local item = find(id) if not (item) then return end`.

> `if not local` works too, and `prefer_const` reports it when the name is never reassigned.

### `Refutable local`

```luau
local Move(x) = cmd else
  return
end
```

Binds a variant's payload. With `else … end`, the block runs instead on any other variant; end it with `return`, `break`, `continue` or an error, so `x` holds after it. Without `else`, another variant raises, naming the pattern and the tag found.

> v0.1.0 does not enforce that the `else` block leaves: one that falls through compiles, and `x` is nil after it.

### `for patterns and filters`

```luau
for _, { x, y } in points where x > 0 do
```

A loop name may be a table or array pattern, destructured on the `do` line, and `where` after the iterator emits `if not (cond) then continue end`. `for` also reads a std `Queue`, `Heap` or `Iter` directly.

## Types

### `Return type`

```luau
function f(x: number) -> string
function g() -> ()
```

The style guide's return-type form. It emits Luau's `: string`.

> `: T` still compiles and no lint reports it; `-> ()` is the recommended spelling for "returns nothing".

### `type`

```luau
type Id = number
type Pair<T> = { first: T, second: T }
```

Luau's alias, passed through. `export type` sends it out; the module table carries no key for it.

### `interface`

```luau
interface Player extends Named
  hp: number
  read id: string
end
```

A structural shape, sugar over `type`. A base joins with `&`: `interface Item extends Named` emits `type Item = Named & { … }`. Fields take `read` and `write`, never `private`.

### `satisfies`

```luau
{ volume = 1 } satisfies Settings
```

Checks a literal against `T` and gives it type `T`. The check artifact reports a key of `T` the literal leaves out.

> An extra key is not reported, since Luau reads a wider table as a subtype. Annotate the binding when the shape must be exact.

### `Type negation`

```luau
local v: ~nil = 1
function f<T: ~nil>(x: T)
```

Every value that is not a `T`. Emits `__neg<T>`, a type function over `types.negationof` declared on line 1. `~` binds tighter than `|` and `&`.

> Luau negates primitives, singletons, classes and unions of them; a table or function type reports. Start with `~nil`.

### `Explicit type arguments`

```luau
f<<number>>(x)
new Stack<<string>>()
import<<Module>>(path)
```

Type arguments at a call, `new` or `import`. The ship artifact drops them at a call or `new`, with no cast, and the check artifact keeps `f<<number>>(x)` for luau-lsp. `import<<T>>` casts the value to `T` in both.

> A single `<` there would parse as a comparison; `single_angle_call` reports it.

### `Array types`

```luau
T[]        -- Array<T>
read T[]   -- ReadArray<T>
write T[]  -- WriteArray<T>
```

`T[]` names the std array with its methods. `ReadArray` has every reader and no writer; `WriteArray` has `push` alone. An `Array<T>` passes where either is asked.

### `Partial / Readonly / Sink`

```luau
Partial<T>   Readonly<T>   Sink<T>
```

Every field of `T` optional, read-only, or write-only: type functions from `@alloy/std/types`, so under `[std] globals = "none"` each needs its import. A file's own `type Partial` replaces it there.

### `Definition files`

```luau
declare function name(params) -> R
declare name: T
declare class Name extends Base … end
```

A `.d.aly` holds global declarations that every file sees. `alloy build` writes nothing for it; the editor and `alloy flux` load every `.d.aly` under `[build] in`.

## Traits and extensions

### `trait`

```luau
trait Describe
  function label(self) -> string
  function name(self) -> string
    return 'item'
  end
end
```

Method signatures, with a body as a default. An impl that omits a defaulted method gets the default copied in.

> A trait takes no type parameters.

### `impl Trait for`

```luau
impl Describe for Sword … end
```

Implements a trait. A missing method is `TraitContract: impl Describe for Sword does not write label` (book 3.6), with the trait's arity checked too.

### `Bounds`

```luau
function largest<T: Ord>(xs: T[]) -> T
function show<T: Display & Debug>(v: T)
```

The argument must have the trait's methods; `&` asks for several. A std shape resolves to the runtime's type, a trait the file declares to its own.

> Bounds go on functions and methods. A struct, enum or interface takes none, since a Luau alias checks no bound.

### `Operator traits`

```luau
Add  Sub  Mul  Div  Eq  Lt  Le
Display  Call  Len  Concat  Drop
```

An impl writes the metamethod after the block: `impl Add` (`add`) sets `__add`, `Eq` (`eq`) `__eq`, `Lt`/`Le` `__lt`/`__le`, `Display` (`to_string`) `__tostring`, `Call` `__call`, `Len` `__len`, `Concat` `__concat`, and `Drop` (`drop`) the `Destroy` that `delete` runs.

### `Std trait shapes`

```luau
import { Display, Ord } from '@alloy/std/traits'
```

`Display`, `Debug`, `Clone`, `Default`, `Eq`, `PartialEq`, `Ord`, `Add`, `Sub`, `Mul`, `Div` (and `Serialize` from `@alloy/std/serde`): table types for bounds and impls. `alloy doc Traits` lists them.

> Language-owned names: they need no import even under `[std] globals = "none"`.

### `Extension methods`

```luau
export impl string
  function shout(self) -> string
    return self:upper()
  end
end
```

Methods on a type you do not own, collected project-wide. The ship artifact dispatches each call through the runtime, `('hi'):shout()` emitting `__alloy.call(('hi'), "shout")`. The check artifact calls a primitive's extension as a function, `__alloy_string.shout(('hi'))`, and keeps the method call on a type such as `Vector3`.

> `__alloy.call` runs the extension for the receiver's kind first, and the receiver's own method only when that kind has none, so an `export impl Vector3` that writes `Cross` replaces Roblox's `Cross`. A struct has no `__name`, so it dispatches as kind `table`. The docs say a foreign impl needs `export`; a single-file compile without it reports nothing, so write `export` and do not rely on a diagnostic.

## Derives and struct attributes

### `@derive`

```luau
@derive(Eq, Ord, Debug, Clone, Default)
struct Point … end
```

Generates methods from the field list, on the struct's emitted lines. On an enum, `Eq`, `PartialEq`, `Clone` and `Debug` derive.

### `Eq / PartialEq`

```luau
@derive(Eq)
```

`__eq` (and `eq`): equal when every field is. A table, array, map or set field compares by contents.

> Without it, `==` on two structs is identity; `identity_compare` warns about that.

### `Ord`

```luau
@derive(Ord)
```

`__lt` and `__le`: fields compare in declaration order, and the first that differs decides, so `<` and `table.sort` work.

### `Debug`

```luau
@derive(Debug)
```

A `debug` method and `__tostring`, both printing the name and fields. On an enum, call `E.debug(v)`.

### `Clone`

```luau
@derive(Clone)
```

`clone()`: a new value with the same metatable. A field that is a struct deriving `Clone` clones through it; an array, table, `HashMap` or `Set` field is copied.

> Anything else, an Instance included, is shared, not cloned. A clone is `==` under `Eq` but not the same table.

### `Default`

```luau
@derive(Default)
-- Stats.default()
```

`Name.default()`: declared defaults kept, the rest at the zero of their type (`0`, `''`, `false`, nil, an empty collection, `Vector3.zero`, `CFrame.identity`…).

> A field with no zero, such as an Instance, reports: give it a default.

### `Serialize / Deserialize`

```luau
import { Serialize, Deserialize } from '@alloy/std/serde'
@derive(Serialize, Deserialize)
```

`to_table()` and `serialize()` write a plain table for a DataStore or a remote; `Name.from_table(t)` reads it back with the metatable and defaults. Nested deriving structs, arrays, `HashMap`s and `Set`s round-trip.

> A missing key leaves its default, so an old save loads. An enum field with an unknown variant raises, naming the struct, field and value.

### `@sealed`

```luau
@sealed
struct Config … end
```

A write to an undeclared key raises `Config has no field volme` at run time (`__newindex` on the class), and the check artifact rejects it. A declared field set to nil still writes.

> `@sealed` stops typos, not writes: mark a field `read` to stop writes to it. It adds a metamethod on absent keys, so keep it off hot-loop structs.

### `@rename / @skip`

```luau
@rename('regen_per_second')
regen: number
@skip
connection: RBXScriptConnection?
```

Serde field attributes: the key a field takes in the derived table, or leave it out entirely. Imported from `@alloy/std/serde`.

### `@alias`

```luau
@alias('hp')
health: number
```

A second key for a field: `p.hp` reads and writes `p.health`'s slot, and `from_table` reads `hp` when `health` is absent. `Serialize` writes the field's own key only.

### `@rename_all`

```luau
@rename_all('camelCase')
```

The case every derived key takes: serde's eight styles, from `lowercase` to `SCREAMING-KEBAB-CASE`. A field's `@rename` wins.

### `@deny_unknown_fields`

```luau
@deny_unknown_fields
```

`from_table` raises on a key the struct neither names nor aliases, so a typo in stored data fails at load.

## Modules

### `Named imports`

```luau
import { a, b as c, type T } from './m'
import * as M from './m'
```

`{ }` picks exports by name and `as` renames; `* as M` takes the whole module, with the default under `M.default`. The `require` runs on the import's own line.

> Replaces `require(…)`, which `raw_require` reports. An import inside a function or `do` block binds only there.

### `Default import`

```luau
import Name from './m'
import M, { a } from './m'
```

`local Name = require('./m').default`. A plain Luau module, a data file, or an Alloy module that ends in `return` has no export table: a bare name takes the returned value.

### `Type-only imports`

```luau
import type { T } from './m'
```

Cost nothing: the build drops the `require` of a line that binds only types, so two modules may import each other's types. The check artifact keeps the line.

### `export`

```luau
export local x = 1
export function f() end
export struct Vec2 … end
export { a, b as c }
export { Shape } from './shapes'
```

Adds a name to the table the module returns. Any declaration takes it. A barrel re-exports with `export { } from`.

> An importer copies the value at require time; a later write to an exported `local` never reaches it (`stale_export`). Export a function, or a table whose fields you write. A top-level `function f()` emits `local function f()`: Alloy has no globals.

### `export default`

```luau
export default expr
export default function make() end
```

One unnamed value under `default`, beside any named exports. The other way to send one value is a final `return expr`; a module that does both reports.

### `Service imports`

```luau
import { Players, RunService } from '@game'
import Players from '@game/Players'
```

Each lowers to `local Players = game:GetService("Players")` on its own line. `'@game'` takes braces and `'@game/X'` a bare name. A longer `@game/…` path is an instance path.

> `@game` is reserved: a mount or Luau alias named `game` is an error. The old `'game:Players'` and `'game'` spellings still read in v0.1.0, with a `game_alias` warning.

### `Std modules`

```luau
import { HashMap, Set } from '@alloy/std/collections'
```

`collections` (HashMap, Set, BitSet, Queue, Heap, Array, Symbol), `iter` (Iter), `result`, `async` (Future, Scope), `signal` (Signal), `traits`, `serde`, `types` (Partial, Readonly, Sink), `roblox` (R15Character, R6Character, Attributes). `@alloy/std` alone holds every name.

> An import writes nothing to the output: the emit reaches every std name through the runtime. Under `[std] globals = "none"` (the default) a missing import is an `ImportError`. `Future`, `Result`, `Ok`, `Err`, `Array` and the traits never need one.

### `Data imports`

```luau
import data from './data.json'
import { coins } from './config.toml'
```

A `.json` or `.toml` file imports as a table. The emit drops the extension, and the build writes the file as a Luau module beside the output.

### `import()`

```luau
local m = import('./m')
import './side-effects'
```

The expression form: the module's default, or its whole value. `import<<T>>(expr)` types a dynamic path; without it the value is `unknown`.

### `namespace`

```luau
namespace Shop
  const TAX = 0.1
  private function helper() … end
  function price(b: number) -> number … end
end
```

One name over a group of declarations; outside, members read as `Shop.price`. Emits `local Shop = {}` and prefixed locals assigned onto it. A type member emits as `Shop_Type`.

> Members are `public` by default; `private` keeps one inside. `export namespace` sends the group with its types.

### `File kinds`

```luau
x.aly   x.alx   x.d.aly   x.luau
```

A module, a module with markup, a declaration file (no output), and plain Luau, which is copied as is. All four require each other.

## Results

### `Result / Ok / Err`

```luau
Result<T, E>
Ok(value)
Err(error, trace?)
```

A function that can fail returns one. Read it with `match`, `if const Ok(v) = r`, or the methods. `Err` carries `trace`, which `try` and `Result.pcall` fill in.

> Language-owned: no import needed. Return an `Err` for an expected outcome; throw with `error` when a correct caller should never get there.

### `Methods`

```luau
r:unwrap()   r:expect(msg)   r:unwrap_or(d)
r:map(f)   r:map_err(f)   r:and_then(f)
r:or_else(f)   r:ok()   r:is_ok()
```

`unwrap` raises an `Err` with its trace; `unwrap_or` gives `T | D`; `map` and `map_err` transform one side; `and_then` chains a fallible step; `ok()` drops the error for `??`.

> `is_ok()` narrows nothing: bind the value with `match` or `if const Ok(v)`.

### `try`

```luau
local x = try parse(a)
```

Unwraps an `Ok`, or returns the `Err` from the enclosing function, which must return a `Result`. Emits `local _1 = parse(a) if _1.tag == "Err" then return _1 end local x = _1._1`. At module top level it returns the `Err` from the chunk.

> A `Result` nobody reads draws `dropped_result`.

### `try do`

```luau
local r = try do
  risky()
end
```

A block whose value is a `Result`: a plain value becomes `Ok`, a `Result` passes through, and a throw becomes `Err` with the traceback. Emits `__alloy.try_block(function(__fail) … end)`.

> The body runs in a function of its own, so `...`, `break` and `continue` inside it are errors, and an inner `try` returns from the block.

### `Result.pcall`

```luau
Result.pcall(f, ...)
```

Calls a function that throws: `Ok` of its return or `Err` of its error, with the traceback. Use it at the boundary with Roblox APIs, then let Results flow inward.

> Replaces `pcall`/`xpcall` (`raw_pcall`).

## Async

### `async function`

```luau
async function load(id: number) -> Profile
```

Callers get a `Future<Profile>`; the declared type is the inner one. The body runs on `task.spawn` under `xpcall`, emitted as `return __alloy.future(function() … end)` around the same lines.

> `-> Future<T>` means the same thing. With no return value it is `Future<nil>`.

### `await`

```luau
local v = await load(1)
```

Yields until the Future settles, then returns its value or rethrows its error. Accepts a Future or anything with `andThen`.

> It needs an async context, or it is `AsyncError` (book 3.3): an `async function` body, an `async do` block, an `after` block, the top level of a `.server.aly`/`.client.aly` script, or a function passed to `task.spawn`/`defer`/`delay` or a signal's `Connect`/`Once`. A plain nested `function` resets the context.

### `async do`

```luau
local job = async do … end
```

An expression whose value is a Future of the block's last line or `return`. As a statement, the block runs for its effects and the Future is dropped.

### `try await`

```luau
local raw = try await store:get(id)
```

A rejection becomes an `Err` returned from the enclosing function. If the awaited async function itself returns a `Result`, that Result is the value, not an `Ok` around it.

### `Future combinators`

```luau
Future.all(fs)   Future.race(fs)   Future.any(fs)
Future.all_settled(fs)   Future.delay(s)
Future.resolve(v)   Future.reject(e)
```

`all` waits for every value (first failure fails it); `race` takes the first to settle; `any` the first to succeed; `all_settled` gives one Result each; `delay` is an awaitable `task.wait`.

### `Future methods`

```luau
f:and_then(on_ok, on_err)   f:cancel()   f:is_settled()
```

`and_then` (also `andThen`) runs a callback on settle and returns the same Future. `cancel` closes the task; an `await` on it raises.

> A rejected Future that nobody awaits keeps its error silently. Await it, or pass `on_err`.

## Cleanup

### `delete`

```luau
delete value
delete t.field
```

Cleans up whatever it is, at run time (`__alloy.delete`): destroys an Instance, disconnects a connection, cancels a thread, calls a function, closes a `Scope`, destroys a `Signal`, or calls a table's `Destroy`/`Disconnect`/`destroy`/`disconnect`. On a field it then sets the slot to nil.

> `delete` takes no timer.

### `destroy`

```luau
destroy part
```

Calls the value's `Destroy` or `destroy` and nothing else. A single-file compile does not check the type in v0.1.0, so a value with neither fails at run time.

> Use `delete` for connections, threads and scopes. On a plain Instance, `prefer_destroy` (style group, `allow` by default; `[lint.rules] prefer_destroy = "warn"` turns it on, `[lint] strict` does not) asks for `destroy part` over `delete part`, since it says the one thing that happens.

### `destroy after`

```luau
destroy part after 2
```

An Instance goes to `game:GetService("Debris"):AddItem(part, 2)`, which outlives the script; another value goes on a `task.delay`.

### `after`

```luau
after 3 do … end
after 3 where alive do … end
```

`task.delay(3, function() … end)`. The `where` condition is read when the timer fires, not when it is scheduled.

> `return` leaves the block, not the enclosing function. Parenthesize a computed delay: `after (n + 1) do`.

### `Scope`

```luau
import { Scope } from '@alloy/std/async'
local scope = new Scope()
scope:add(part.Touched:Connect(f))
delete scope
```

A cleanup bag. `add` takes anything `delete` accepts and returns it; `clean` (or `delete scope`) runs the cleanups newest first; `extend` makes a child scope.

> `scope:add(conn, part)` names an owner Instance: `delete part` then disconnects the item before the Destroy.

### `Signal`

```luau
import { Signal } from '@alloy/std/signal'
local damaged = new Signal<<Player, number>>()
```

A typed signal with `RBXScriptSignal`'s shape: `Connect`, `Once`, `Wait`, `Fire`, `FireDeferred`, `DisconnectAll`, `Destroy`, each with a snake_case twin (`connect`, `fire`…).

> `Signal.wrap(source)` and `Signal.collect(source)` adapt any signal, a Roblox one included.

## Collections

From `@alloy/std/collections` and `@alloy/std/iter`. The style guide prefers these to `{ [K]: V }` tables for keyed data.

### `HashMap`

```luau
local m: HashMap<string, number> = new HashMap()
$map[['sword', 10], ['pet', 25]]
```

`get` (returns `V?`), `set` (chains), `has`, `remove`, `len` (counted, O(1)), `keys`, `values`, `entries`, `get_or_insert`, `clear`. `for k, v in m` walks the pairs.

> An empty map infers nothing: annotate the binding.

### `Set`

```luau
local s: Set<string> = new Set()
$set[1, 2, 3]
```

`add` (chains), `has`, `remove`, `len`, `union`, `intersection`, `difference`, `to_array`. `for v in s` walks the members in no set order.

### `BitSet`

```luau
new BitSet(1024)
```

A set of whole numbers from 0, one bit each in a buffer: about 2x faster than a `Set<number>` for dense ids. Walks members in rising order.

### `Queue`

```luau
local q: Queue<Job> = new Queue()
```

FIFO over a ring of indices: `push`, `pop`, `peek`, `len`, `is_empty`. O(1) at both ends.

### `Heap`

```luau
new Heap(function(a, b) return a.cost < b.cost end)
```

A binary heap; `pop` yields the least value under `less`, which defaults to `<`.

### `Array methods`

```luau
xs:push(…)   xs:pop()   xs:map(f)   xs:filter(f)
xs:find(f)   xs:reduce(f, init)   xs:sort_by(less)   xs:join(sep)
```

Every `[ ]` literal and `T[]` carries them. `Array.from(t)` adds the metatable to a plain table in place.

> Annotate a `reduce` accumulator when its body uses it, and expect a third chained `map` to type as `any`.

### `Iter`

```luau
import { Iter } from '@alloy/std/iter'
Iter.from(xs):filter(f):map(g):take(5):collect()
Iter.range(1, 10, 2)
```

A lazy iterator over an array, a function, or a std collection: nothing runs until `collect`, `for_each`, a reducer or a `for` loop pulls.

### `Symbol`

```luau
new Symbol('name')
```

A unique frozen key no string can collide with; it prints as `Symbol(name)`.

## Attributes and macros

### `attribute`

```luau
attribute icon(asset: string = 'none') on struct, variant
```

Declares metadata the compiler checks and the runtime reads. Targets: function, struct, enum, variant, field, param, remote, message, interface, type, local, namespace, impl, trait. A parameter may carry a default.

> Using one on a target it does not list is an `AttributeError` (book 3.11).

### `Contracts`

```luau
attribute service on impl as
  requires public function Start(self)
  requires private field state: number
end
```

What the thing the attribute sits on must carry. Checked where it is used and emits nothing; a miss is `AttributeContract: @service requires a public function Start(self); S declares none`.

> Structural, like an interface. A struct and its impl blocks count as one type.

### `each`

```luau
requires private function each lifecycles(self)
```

One `requires` clause per entry of a list argument: `@provider([ Lifecycle.Init, Lifecycle.Start ])` asks for `Init` and `Start`.

### `Attributes`

```luau
import { Attributes } from '@alloy/std/roblox'
Attributes.get(Sword, icon)
```

Reads declared metadata at run time. Also `field`, `variant`, `fields` and `of`. The second argument is the declared name, not a string.

> One parameter reads as its value, several as a table, none as `true`.

### `@allow`

```luau
@allow(too_many_arguments)
@allow(flux.unused_variable, luau.LocalShadow)
```

Quiets named lints over the declaration or statement it sits on, and moves with the code. A name is a lint, a group, or a tool-prefixed lint (`flux.`, `luau.`, `alx.`, an ingot's name).

> A name no lint has is an error, and an error kind cannot be allowed: use `--@alloy-expect-error` for that.

### `@deprecated`

```luau
@deprecated('renamed to mix')
@deprecated({ use = 'mix', reason = 'renamed' })
```

Luau's own: a call to the function is a lint. Emits `@[deprecated {reason = …}]`.

### `Luau pass-through`

```luau
@native   @checked   @inline   @noinline
@[native, deprecated]
```

Luau's own attributes, written to the emit as they are, so a Luau file stays valid Alloy.

### `macro`

```luau
macro clamp01(x)
  math.clamp(x, 0, 1)
end
$clamp01(2)
```

A compile-time template called as `$name(…)`. Every `local` in the body is renamed per expansion; parameters take defaults and `...`. The declaration's lines go blank in the emit.

> In its own file the body may read a module local. An exported macro does not carry its file's locals: where it expands, the name reads as a global, with no report. Pass values in. A macro that calls itself is a `MacroError` ("expands itself"); nesting stops at depth 16.

### `export macro`

```luau
export macro logit(x) print(x) end
import { logit } from './mac'
```

Sends a macro to other files. A macro is source, not a value: nothing lands in the module table.

### `Debug intrinsics`

```luau
$dbg(expr)   $todo('why')   $unreachable()
```

`$dbg` prints `file:line: <source> = <value>` and returns the value. `$todo` and `$unreachable` raise with the file and line.

### `Source intrinsics`

```luau
$nameof(a.b.c)   $stringify(expr)
```

Compile-time strings: `"c"`, and the expression's source text.

### `Collection intrinsics`

```luau
$set[1, 2]   $map[['a', 1]]   $bnot(x)
```

`Set.from({ 1, 2 })`, `HashMap.from({ ['a'] = 1 })`, and `bit32.bnot(x)`. The closed set of intrinsics also holds `$assert`, `$assert_eq`, `$expect` and `$matches`.

## Roblox

### `@cfg`

```luau
@cfg(server)
local function start_server() -> ()
  …
end
```

Code for one side, checked at run time through RunService (`__alloy.cfg.server()`); nothing is removed at build time. A function with no value returns at once on the wrong side; a function that returns a value raises (`key is @cfg(server) and cannot run here`).

> On a statement it is a guard on the same line. A `@cfg` local is nil on the other side. Replaces `RunService:IsServer()` checks.

### `@cfg conditions`

```luau
server  client  studio  edit  running  test
not  and  or  any(…)  all(…)
```

`test` is true in an `alloy test` run. Two `@cfg` lines over one statement join with `and`.

### `Script sides`

```luau
main.server.aly   main.client.aly
```

Builds a Script or a LocalScript. The name also says which half of a `remote` the file sees; any other file is shared and sees both.

### `remote`

```luau
remote Damage(target: Player, @u16 amount: number) from client
remote function GetCoins() -> number from client
```

A typed channel. `from client`, `from server`, or `from client or server` names who fires, or who calls a `remote function`: `GetCoins` above is the client asking the server, and `from server` would make the server the caller. Emits `__alloy.remote({ name, kind, from_client, from_server, params, defaults, attrs, wire })` on the same line.

> Replaces `Instance.new('RemoteEvent')` (style guide). Parameters may take defaults, but not patterns.

### `The remote object`

```luau
Damage.fire(…)   Damage.fire_all(…)   Damage.fire_except(p, …)
Damage.on(handler)   Damage.once(handler)   Damage.wait()
GetCoins.call()   .on_ratelimited(h)   .spec   .instance
```

From the server, `fire` sends to the player passed first. `call` on a remote function yields a `Future<R>`. A server handler gets the sender first: `Damage.on(function(sender, target, amount) … end)`.

### `Remote attributes`

```luau
@unreliable   @immediate   @ratelimit(5, 1)
@timeout(10)   @validate(fn)
```

An UnreliableRemoteEvent (payload limit 900 bytes); skip the per-frame batch; a per-player token bucket on the server; reject an unanswered `call` after N seconds; a server predicate `fn(sender, ...)` that drops a bad event before the handler.

### `Wire widths`

```luau
@u8  @u16  @u32  @i8  @i16  @i32  @f32
```

The size of a `number` parameter or struct field on the wire. Without one a number is 64 bits.

> A value that does not fit raises at the fire: `Heal: argument 1: 300 does not fit @u8`.

### `Wire layout`

```luau
@wire(buffer)   @wire(table)
```

The compiler picks a buffer or a table, whichever sends fewer bytes; `@wire` forces one. Structs, records and arrays pack field by field; enums cross; Instances and datatypes travel beside the buffer. The receiver checks every packet and drops one that does not fit.

### `Wire contract`

```luau
error(4.3): WireType
```

A remote parameter cannot be a function, thread, Future or Signal: `parameter cb has type () -> (), which is a function type; a remote carries only data`.

### `parallel do`

```luau
parallel do
  …
end
```

Emits `task.desynchronize() do` on the header's line and `task.synchronize() end` on the `end`'s, in an Actor script or a module one requires.

> `ParallelError` refuses what this file shows: Instance property and `Parent` writes, `new Instance`, `Destroy`, `Clone`, `destroy`, `delete`, `await`, a nested `parallel`, and a `return`/`break`/`continue` that leaves. A guard rail, not a proof.

### `Actor scripts`

```luau
physics.server.actor.aly
```

Builds an `Actor` named `physics` holding the script, with an `init.meta.json`. One file makes one Actor; clone it at run time for a pool.

> Only a script takes `.actor.`, and it goes after the side: `x.actor.server.aly` reports the order.

### `message`

```luau
export message Step(dt: number, gravity: number)
Step.fire(worker, 0.016, 196.2)
Step.on(function(dt, gravity) … end)
```

A typed actor message: `fire` sends to an Actor, `on`/`once` bind on the calling script's Actor (which needs `.actor.`). `as parallel` binds with `BindToMessageParallel`.

> Arguments cross without a serializer, so a struct, payload enum or `HashMap` does not arrive whole: each is a `WireType` report.

### `reply`

```luau
export message Light(job: number, input: buffer) reply(job: number, levels: buffer)
Light.replied(function(job, levels) … end)
```

An answer channel: the `on` handler gets `respond` after its parameters, and `replied` binds on any script. Nothing matches a reply to its request, so pass a job id both ways.

### `R15Character / R6Character`

```luau
import { R6Character } from '@alloy/std/roblox'
```

Typed character models with every part optional. `Player.Character` types as `R15Character?`, or `R6Character?` under `[roblox] rig = "R6"`.

## Tests

### `@test`

```luau
@test
function clamps_high()
  $assert_eq(clamp01(2), 1)
end
```

A test beside the code. The ship artifact blanks it line for line; the check artifact keeps it typed and registers it with the runtime. An `async` test is awaited.

### `@test namespace`

```luau
@test
namespace SatchelTests … end
```

Every public function of the group is a test named `SatchelTests.name`; a `private` member is a helper.

### `$assert`

```luau
$assert(cond)
```

`assert(cond, "assertion failed: <source text>")`.

### `$assert_eq`

```luau
$assert_eq(a, b)
```

Raises with both source texts and both values when they differ.

### `$expect`

```luau
$expect(hp):toBe(100)
$expect(f):toThrow('locked')
```

Lest's jest-style matchers, typed from the value.

> Only inside a `@test` function or namespace (else `TestError`, book 3.14), and only with `[test] lest = true`.

### `Specs`

```luau
src/inventory.aly -> tests/inventory.spec.luau
```

`alloy test` writes one lest spec per source with a `@test`, holding the tests and the top-level code they reach. The first run with `lest = true` writes `lest.toml` and adds `lest = ".lest/core"` to the Luau config's aliases.

> `alloy test file.alx` is refused; a project run does write `.alx` specs.

## The alloy CLI

An unknown `--flag` is an error naming the command, never ignored.

### `alloy build`

```luau
alloy build [file] [-W|--watch] [--out <dir>]
  [--config <file>] [--check] [--map] [--wait-timeout <s>]
```

Compiles every `.aly` and `.alx` under `[build] in` to `[build] out`, mirroring the tree, and writes the runtime as `alloy.luau` at the output root. With a file, prints it to stdout. `--check` emits the check artifact.

> A file with an error is not written; its last output stays. The build runs no lints. `--map` works on a single file only; a project build ignores it.

### `alloy check`

```luau
alloy check [file…] [--fix] [--strict] [--deny-warnings]
  [-W|-A|-D <lint>] [--config <file>]
```

Compiles everything and writes nothing; reports diagnostics and lints at their levels, and exits 1 on any diagnostic or denied lint.

> `--fix` and the `-W/-A/-D` levels apply only when files are named; a project-wide `check` does not read them. Use `alloy flux --fix`.

### `alloy lint`

```luau
alloy lint [file…] [--fix] [-W|-A|-D <name>]
  [--strict] [--deny-warnings] [--list] [--config <file>]
```

The lints alone, in eight groups: correctness, suspicious, style, complexity, perf, roblox, pedantic, naming. `--list` prints every lint and level in the `[lint.rules]` form.

### `alloy flux`

```luau
alloy flux [file] [--fix] [--watch] [--explain <lint>]
  [--no-typecheck] [-W|-A|-D <name>] [--list] [--strict] [--deny-warnings]
```

The whole analysis: compile, luau-lsp over the check artifact with type errors mapped onto Alloy lines, and every lint, including luau-lsp's under `luau`. Also reports `circular_import`.

> `--fix` applies only rewrites that keep the program the same. luau-lsp is found on PATH, then in `~/.alloy/bin` and `~/.ember/bin`.

### `alloy fmt`

```luau
alloy fmt [paths] [--check] [--config <file>]
```

Anneal formats `.aly` and `.alx` in place from the tokens and `[fmt]`. Statements keep their lines. `--check` writes nothing and fails when a file would change.

### `alloy test`

```luau
alloy test [file] [--run] [--coverage] [--filter <text>]
  [-W|--watch] [--check] [--out <dir>] [-- <lest args>]
```

Builds, then writes the lest specs. `--run` runs lest; `--filter` runs tests whose name holds the text; `--check` fails when a spec would change. With a file, prints its spec.

### `alloy doc`

```luau
alloy doc [topic] [--json]
alloy doc 4.2   alloy doc ExhaustiveMatch
alloy doc style   alloy doc HashMap:get
```

Explains a keyword, operator, intrinsic, attribute, std name, lint, article, book section or error kind. No topic lists them all; `--json` prints everything.

### `alloy init`

```luau
alloy init [-y|--yes] [-i|--interactive] [-n|--non-interactive]
```

In a terminal, a wizard: "Use the recommended setup?" writes a game (src/client, server, shared and their `[mount]` entries), ember, lest, the `temper` preset, strict mode and an AGENTS.md. `--yes` writes the same with no prompt. A pipe, a script, or `--non-interactive` writes only the plain template.

> It always writes `alloy.toml` and `.alloy/alloy.schema.json`, plus a `.config.luau` (strict, `alloy` alias) only when neither Luau config exists; otherwise it adds the mode and alias to the existing one. It never writes a `.luaurc`, and fails if `alloy.toml` exists.

### `alloy migrate`

```luau
alloy migrate
```

Rewrites `alloy.toml` as `.config.aly`, keeping keys and comments, then removes `alloy.toml`. It writes nothing if the loaded configs would differ.

### `alloy mcp`

```luau
alloy mcp
claude mcp add alloy -- alloy mcp
```

Serves the docs to a coding agent over MCP (stdio JSON-RPC 2.0). Tools: `search_docs`, `read_doc`, `list_topics`, `style_guide`.

### `alloy self`

```luau
alloy self install | update [--version <v>] | uninstall
alloy self code [--dry-run] | schema   [--dir <path>]
```

`install` copies `alloy` and `alloy-lsp` to `~/.alloy/bin`, fetching what is missing from the GitHub release. `update` installs a release. `code` points VS Code and Zed at the `alloy.toml` schema; `schema` prints it.

> The subcommand is `uninstall`, though the help line says "remove".

### `alloy ingot`

```luau
alloy ingot install [name] | update [name]
alloy ingot new <name> | info <dir> | run <dir> <file>
```

`install` fetches declared ingots the project lacks; `update` re-resolves every unpinned one. `new` scaffolds a Rust ingot, `info` prints a manifest, and `run` pushes one file through (`--lint`, `--output`, `--format`, `--hover <byte>`, `--complete <byte>`).

> A build never fetches: a missing ingot is a report naming `alloy ingot install`.

## alloy.toml

Every key has a default and an unknown key is an error. Values below are the source defaults; `alloy init` changes only `wait_timeout`.

### `[build]`

```luau
[build]
in = "src"
out = "build"
exclude = []
clean = false
artifact = "ship"
```

The source and output roots, globs to skip, and the artifact: `ship` runs on Roblox, `check` keeps types for luau-lsp.

> `clean = true` also deletes every `.luau` under `out` that no source makes; off, the build removes only its own stale outputs (listed in `.alloy/outputs.txt`).

### `[emit]`

```luau
[emit]
wait_timeout = 5
# std_require = "@alloy"
```

`wait_timeout` (unset by default) is passed to every `WaitForChild` that `=>` emits. `std_require` is the string the emit requires the runtime by.

> A project build defaults `std_require` to a relative path, or to the runtime's instance path under a mount, not `@alloy`. `erase_type_imports` is read by nothing.

### `[std]`

```luau
[std]
globals = "none"   # "all", or ["Signal", "Iter"]
```

Which std names a file may use without an import. Language-owned names never need one.

### `[fmt]`

```luau
[fmt]
recommended = true
column_width = 100
indent_type = "spaces"
indent_width = 2
quote_style = "force-single"
prefer_const = true
```

Anneal's layout. Also `line_endings`, `leading_zero`, `call_parentheses`, `space_after_function_names`, `collapse_simple_statement`, `block_newline_gaps`, `magic_trailing_comma`, `trailing_comma`, the `space_inside_*` keys, `align_struct_fields`, `expand_imports`, `exclude` and `fix_naming`.

> `recommended = false` preserves each file's layout and applies only the keys you set. `prefer_const` rewrites never-reassigned locals to `const`; the docs' table omits it.

### `[fmt.*] tables`

```luau
[fmt.call_chains]
style = "preserve"
min_calls = 3

[fmt.sort_requires]
enabled = false

[fmt.alx]
attribute_quotes = "double"
```

Method-chain breaking, import sorting (`grouping = "flat"` or `"by-kind"`), and markup layout (`bracket_same_line`, `attribute_per_line`, `self_closing_space`, `text_wrap`, `blank_lines`).

### `[lint]`

```luau
[lint]
recommended = true
strict = true
```

`recommended` applies each lint's declared level (off: every lint starts at `allow`). `strict` turns the pedantic group on at `warn`.

> The old `deny`, `warn` and `allow` lists still read in v0.1.0, with a warning naming `[lint.rules]`.

### `[lint.rules]`

```luau
[lint.rules]
correctness = "deny"
raw_require = "allow"
"tailwind/unknown_class" = "warn"
alx.static_conditional_child = "deny"
```

A lint, a group, an ingot's `<ingot>/<lint>`, or `alx.<name>`, at `allow`, `warn` or `deny`. A name beats its group; `luau` is luau-lsp's group.

### `[lint.naming]`

```luau
[lint.naming]
const = "SCREAMING_SNAKE_CASE"
variable = ["snake_case", "camelCase"]
```

The case of each kind of name for `naming_convention`: `snake_case` for variables, functions, methods, fields, attributes and macros; `PascalCase` for types, structs, enums, variants, traits, namespaces, remotes and components.

### `[flux]`

```luau
[flux]
typecheck = true
definitions = []
roblox_types = true
security_level = "PluginSecurity"
new_solver = true
```

What `alloy flux` runs: luau-lsp (`luau_lsp` names the binary), extra definition files, the Roblox globals, and the solver. Also the complexity thresholds `too_many_arguments` (7), `too_many_lines` (100), `max_nesting` (5) and `cognitive_complexity` (25).

> `new_solver = false` runs the old solver; the docs' table omits the key.

### `[roblox]`

```luau
[roblox]
rig = "R15"   # or "R6"
```

The character type `Player.Character` reads as. The editor's `alloy-luau.rig` setting wins.

### `[test]`

```luau
[test]
out = "tests"
suite = "alloy"
lest = true
shim = true
```

Where specs go, the suite name in `lest.toml`, whether to write lest's config and alias, and `shim`: engine doubles (`Vector3`, `task`, `game`…) loaded before each spec so shared code runs on a plain VM.

### `[project]`

```luau
[project]
name = "game"
runtime = "@game/ReplicatedStorage/Alloy"
sourcemap = true
source_of_truth = true
mount_aliases = true
```

Also `file` (which Rojo project to read). `source_of_truth` writes `default.project.json` and `.alloy/build.project.json` from `[mount]`; `mount_aliases` serves mount names as aliases; `sourcemap` writes `sourcemap.json` on each build.

> With a placed `[mount]` table and `source_of_truth` on, each build writes `default.project.json` from the table, over any file already there. Without a table Alloy reads the root's project file and never writes it. The wizard writes `runtime`; it is the default either way.

### `[mount]`

```luau
[mount]
shared = ["src/shared", "@game/ReplicatedStorage/Shared"]
types = "src/shared/types"
```

Where each folder lands in the DataModel, for tools that read no project file. A bare path is an alias for a folder another mount already carries.

> `game` and `alloy` are reserved. A name in `.config.luau` or `.luaurc` wins over a mount name.

### `[ingots]`

```luau
[ingots]
tailwind = "ingots/tailwind"
logger = { repo = "someone/logger-ingot" }
sorter = { repo = "someone/sorter-ingot", version = "0.2.0" }
```

A path, or a GitHub release. A missing `version` is `^`: install resolves it to the latest release and records it in `.alloy/ingots.lock`. A `version` pins. The table form also takes `path`, `asset`, `order` and `lints`.

> The lock holds only while the store has that release. The store (`.alloy/ingots/`) is gitignored, so on a fresh clone `alloy ingot install` fetches an unpinned ingot's latest release, whatever the lock says. Pin a `version` for repeatable builds.

### `[ingot.<name>]`

```luau
[ingot.tailwind]
sort_classes = true
```

One ingot's own options, over its manifest's defaults. The compiler passes them through unread.

### `[alx]`

```luau
[alx.factory]
backend = "table"
create = "create"

[alx.elements]
[alx.properties]
```

Markup lowering, in `luaux.toml`'s shape: `backend`, `create`, `children`, `event`, `compute`, `use`, `fragment`, `interpolate`, `merge`. With no `[alx]`, markup lowers to `React.createElement`.

> A `luaux.toml` beside the file is still read while `[alx]` is empty.

### `.config.aly`

```luau
export const build = { out = 'dist' }
export const fmt = { indent_width = 4 }
```

The same keys in Alloy, as exports, a default export, or a `return`. It runs on load (two-second limit). A folder with both files reads `alloy.toml`.

### `.config.luau / .luaurc`

```luau
return { luau = { languagemode = 'strict', aliases = { alloy = './build/alloy' } } }
```

Luau's own config, the source of the project's aliases. `.config.luau` wins over `.luaurc`. A `--!nonstrict` line still wins for one file.

## Directives

Comments that steer one file's diagnostics: the compiler's, the lints and the checker's.

### `--@alloy-ignore`

```luau
--@alloy-ignore <reason>
local x = y.z --@alloy-ignore
```

Silences every diagnostic on the next code line, or on its own line at the end of one.

### `--@alloy-ignore-start / -end`

```luau
--@alloy-ignore-start prefer_const
…
--@alloy-ignore-end
```

Silences a region, optionally for one lint or checker kind. Pairs nest.

> A start with no end, or an end with no start, is a `DirectiveError`.

### `--@alloy-expect-error`

```luau
--@alloy-expect-error <reason>
```

Silences a line that must hold an error, and is itself an error once the line comes clean.

> Without a reason it draws `missing_reason`. Under `alloy check`, a stale one is settled by `alloy flux`, which runs the type check.

### `--@alloy-nocheck`

```luau
--@alloy-nocheck
```

Silences the whole file, from anywhere in it. For generated or vendored code.

### `--@alloy-lint`

```luau
--@alloy-lint raw_require=allow, explicit_any=deny
```

A lint's or group's level for this file, over `alloy.toml`. The last one in the file decides.

### `--@alloy-preserve`

```luau
--@alloy-preserve <reason>
```

Keeps `alloy flux --fix` and quick fixes off the next line. The lint still reports.

### `Directive or @allow`

```luau
@allow(lint)   vs   --@alloy-…
```

`@allow` for one lint wrong about one declaration; a directive for a line, a region, a whole file, or an error rather than a lint.

> Luau's `--!strict`, `--!nonstrict` and `--!nocheck` pass through. An unknown `--@alloy-` name is a `DirectiveError` (book 4.4) listing the seven.

## Diagnostics

A diagnostic's code is the book section that explains it: `alloy doc 4.2`. Lints are separate: they never stop a build.

### `Reading one`

```luau
src/game.aly:12:5 error(4.2): ExhaustiveMatch: this match is not exhaustive: …
```

Location, level and section, then the kind and the message. The editor shows the same code as `Alloy(4.2)`, linked to the section.

### `3.2`

```luau
ImportError   UnknownModule
```

Modules: a missing std import, a bad import form, a `global` declaration, a path that resolves to nothing.

### `3.3`

```luau
AsyncError
```

An `await` outside an async context.

### `3.4`

```luau
EnumError
```

An enum declared or used wrongly.

### `3.6`

```luau
StructError   ConstructorError   TraitContract   BoundError
```

Unknown field, construction without `new` or with a missing field, a trait method not written, a bound not met. `class` reports here too.

### `3.10 / 3.11`

```luau
MacroError   AttributeError   AttributeContract
```

Macro recursion or misuse; an attribute on a target it does not list; a contract clause not met.

### `3.13 / 3.14 / 3.15`

```luau
MarkupError   TestError   ParallelError   ActorError
```

Markup; `$expect` outside `@test`; a refused statement in `parallel do`; a misplaced `.actor.` or message binding.

### `4.1`

```luau
AlloyError   ResultError   InternalError
```

The general contracts, such as `in` on a `{ }` literal; Result misuse; a compiler bug.

### `4.2 / 4.3 / 4.4`

```luau
ExhaustiveMatch   WireType   DirectiveError
```

A `match` missing a variant; a non-data remote or message parameter; a bad directive.

### `5.x`

```luau
LuauLimit   TypeError   DataError   IngotError
```

A Luau compiler limit (5.1); a luau-lsp type error under `alloy flux` (5.4); a bad data import (5.11); an ingot failure (5.12).

### `6.1`

```luau
SyntaxError   ReservedWord   ConstError   DeclareError   DuplicateError
```

Parse errors, reserved words, a `const` reassigned, a bad `declare`, a duplicate declaration.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [alloy](https://github.com/alloy-luau/alloy) | The compiler and CLI (crate `alloy-luau`) | 0.1.0 | `cargo install alloy-luau alloy-luau-lsp` |
| [alloy-lsp](https://github.com/alloy-luau/alloy/tree/v0.1.0/crates/alloy-lsp) | Language server: a proxy over luau-lsp that desugars each buffer in memory (crate `alloy-luau-lsp`) | 0.1.0 | `cargo install alloy-luau-lsp` |
| [Alloy for VS Code](https://github.com/alloy-luau/extensions) | Editor extension: starts `alloy-lsp` from PATH | — | `github.com/alloy-luau/extensions` |
| [alloy-ingot](https://github.com/alloy-luau/alloy/tree/v0.1.0/crates/alloy-ingot) | The Rust crate for writing ingots (Alloy extensions) | 0.1.0 | `alloy ingot new <name>` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | The type checker behind `alloy flux` and `alloy-lsp` | 1.69.0 | `rokit add JohnnyMorganz/luau-lsp` |
| [lest](https://github.com/lest-luau/lest) | The test runner `alloy test --run` calls | 0.6.0 | `lpm tool add lest && lpm i` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Syncs the build output; reads `.alloy/build.project.json` | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [examples](https://github.com/alloy-luau/examples) | Example Alloy projects | — | `git clone https://github.com/alloy-luau/examples` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`global` declarations.** Removed in v0.1.0: `global local x` is an `ImportError` naming the `export local` and the `import` line that replace it. The `--@alloy-side` and `--@alloy-file-side` directives and the `[contexts]` table went with it; a `.server.aly` or `.client.aly` file name says the side.
- **`import Players from "game:Players"` and `from "game"`.** Write `import Players from '@game/Players'` or `import { Players } from '@game'`. The old paths still read in v0.1.0 with a `game_alias` warning, and the docs call it the last release that reads them.
- **`case n and n > 5`.** A match guard is `case n where n > 5`; `match_guard_and` rewrites the old form.
- **`[lint] deny`, `warn` and `allow` lists.** Write `[lint.rules] name = "deny"`. The lists still read, with a warning.
- **`[alx.lints] static_conditional_child` and a `luaux.toml` beside the file.** Write `[lint.rules] alx.static_conditional_child`, and put the markup settings in `alloy.toml`'s `[alx]` table. A `luaux.toml` is still read while `[alx]` is empty.
- **`[emit] erase_type_imports`.** It does nothing: type-only imports are always dropped.
- **The lints `camel_case_name`, `type_case` and `pascal_case_function`.** All three are now `naming_convention`; the old names still work in `@allow`, `--@alloy-lint` and `[lint.rules]`.
- **`Vector3.new(…)` and `Instance.new(class, parent)`.** Write `new Vector3(…)` (`prefer_new`) and the initializer `new Instance('Part') { Parent = parent }`, which sets `Parent` last (`instance_new_parent`).
- **`local` for a binding that is never reassigned or written into.** Write `const` (`prefer_const`; `[fmt] prefer_const` makes `alloy fmt` rewrite it).
- **`pairs(t)` and `ipairs(t)` in a `for`.** Write `for k, v in t do` (`legacy_iterator`).
- **`require(…)`.** Write `import` (`raw_require`).
- **`X.__index = X` classes.** Write a `struct` and an `impl` (`manual_class`).
- **`pcall` and `xpcall`.** Write `try do … end`, return a `Result`, or call `Result.pcall` at a boundary (`raw_pcall`).
- **`RunService:IsServer()` and `IsClient()` checks.** Write `@cfg(server)` or `@cfg(client)` (style guide).
- **`Instance.new('RemoteEvent')`.** Declare a `remote` (style guide).
- **`wait`, `spawn`, `delay` and `unpack`; lowercase `:connect`, `:wait`, `:remove` and `:clone`; body movers.** Use the `task` library and `table.unpack` (`deprecated_global`), the PascalCase methods (`deprecated_method`), and constraints (`deprecated_body_mover`).
- **`a and a.b`, `a:FindFirstChild('b')`, `if x == nil then x = v end`.** Write `a?.b`, `a->b` and `x ??= v` (`manual_safe_access`, `manual_child_lookup`, `manual_coalesce`).
- **The site tour's emitted Luau.** Its struct, payload-enum, `match` and `remote` emits predate v0.1.0. A v0.1.0 struct adds `__tostring`, a `match` expression ends with an error branch, and a remote emits `{ name, kind, from_client, from_server, params, defaults, attrs, wire }`.
- **"`alloy init` writes `alloy.toml`, `.luaurc`, `.config.luau`"** (README). It writes `alloy.toml`, `.alloy/alloy.schema.json`, and a `.config.luau` only when no Luau config exists. It never writes a `.luaurc`.
- **"The init wizard refuses with `NO_COLOR` set"** (docs). v0.1.0 needs only a terminal: `NO_COLOR` still gets the questions.
- **`alloy self remove`.** The subcommand is `alloy self uninstall`.
