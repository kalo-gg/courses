# Blink reference

*Blink 0.18.8, released April 11, 2026. Checked on September 15, 2026.*

---

## Setup and the CLI

Blink is a standalone compiler, not a package you require. Install the binary with a toolchain manager, or write your definitions inside Studio with the plugin.

### `rokit add 1Axen/blink`

```luau
rokit add 1Axen/blink
rokit update 1Axen/blink
```

The install the documentation recommends. Rokit pins the version in `rokit.toml` and puts `blink` on your PATH inside the project, so everyone on the team and CI run the same compiler.

> GitHub owners are case-insensitive, so `1axen/blink` resolves too; the docs spell it `1Axen`.

### `pesde add --dev 1axen/blink`

```luau
pesde add --dev 1axen/blink
pesde install
```

The other supported install. `pesde.toml` publishes `1axen/blink` with `target.environment = "lune"` and `bin = "release/blink_pesde.luau"` — the compiler is Luau, so pesde runs it through Lune rather than shipping a native binary.

> Official pesde releases start at 0.18.5. Earlier versions are GitHub Releases or Rokit only.

### `Blink Editor (Studio plugin)`

```luau
Creator Store asset 77231976488966
```

Writes, analyses and generates definitions inside Studio, with no external tooling. It asks for script-injection permission on first open because generating means writing ModuleScripts into your place.

> `plugin/src/` builds from the same `src/` compiler, so the language is identical. Its imports only resolve sibling `./` paths.

### `blink <file>`

```luau
blink crosswire            # tries crosswire, crosswire.txt, crosswire.blink
blink src/net/crosswire.blink
```

Compiles one definition and writes the server and client modules. `CLI/Utility/GetDefinitionFilePath` tries the extensions `""`, `.txt` and `.blink` in that order, relative to the path you gave, so the extension is optional.

> `ServerOutput` and `ClientOutput` are both asserted before anything is generated: a file without them fails with "A client output path must be defined."

### `--watch, -w`

```luau
blink crosswire --watch
```

Recompiles whenever the target file or any file it imports changes — imports of imports included. The watcher is the one part of the CLI that uses Lune's `task` and `datetime` modules.

### `-h, -v, -y, -q, -c, --ast, -S`

```luau
-h --help     -v --version   -y --yes      # accept every prompt
-q --quiet    -c --compact                  # silence / compact diagnostics
--ast         -S --stats                    # hidden: dump the AST / timings
```

The whole option set, from `ARGUMENTS` in `src/CLI/init.luau`. `--yes` matters in CI: without it the compiler prompts before creating an output directory that does not exist. `--ast` and `--stats` are marked `Hidden` and do not appear in `--help`.

> `--ast` prints the options table and the declarations and then stops — nothing is written.

## Options

`option Name = Value` at the very top of the file. All thirteen names live in one table in `src/Parser.luau`; an unknown one is `[E2003]` and lists them all back at you. An option after the first declaration is `[E3016]`.

### `ServerOutput`

```luau
option ServerOutput = "../Network/Server"
```

Where the server module is written, relative to the definition file unless the path is absolute. The extension is stripped and `.luau` appended, so `"../Network/Server"` and `"../Network/Server.luau"` produce the same file.

> Required. There is no default.

### `ClientOutput`

```luau
option ClientOutput = "../Network/Client"
```

The same for the client module. Required; the compiler asserts on both output paths before it generates anything.

### `TypesOutput`

```luau
option TypesOutput = "../Network/Types.luau"
```

Optional third file holding just the `export type` declarations, for code that wants the types without requiring either half of the network layer.

> Unlike the other two, this path is used verbatim — it is not run through the path parser, so write the `.luau` yourself.

### `Casing`

```luau
option Casing = Pascal   -- default
option Casing = Camel
option Casing = Snake
```

Renames every generated method. Pascal gives `Fire`/`FireAll`/`FireList`/`FireExcept`/`On`/`Invoke`/`Iter`/`Next`/`Read`/`Write`/`StepReplication`; Camel gives `fire`/`fireAll`/…/`stepReplication`; Snake gives `fire`/`fire_all`/`fire_list`/`fire_except`/`step_replication`.

> Your own names are untouched — casing applies to Blink's methods only. The value is an identifier, not a string.

### `Typescript`

```luau
option Typescript = true   -- default false
```

Also writes a `.d.ts` beside each generated module, for roblox-ts. An output named `init` is emitted as `index.d.ts`.

### `UsePolling`

```luau
option UsePolling = true   -- default false
```

Forces every event in the file to the polling API, as if each had `Call: Polling`. Handy when an ECS schedule drains all incoming traffic in one phase.

> It only affects receiving. The sending side still gets `Fire` and friends.

### `ManualReplication`

```luau
option ManualReplication = true   -- default false
```

Stops the generated module connecting `StepReplication` to `RunService.Heartbeat`, so you decide when a frame's buffered traffic is flushed.

> It does not add `StepReplication` — that is exported either way. The docs say the function is "exposed instead"; the source returns it unconditionally and only the Heartbeat connection is conditional.

### `SyncValidation`

```luau
option SyncValidation = false   -- default true
```

When on, a `SingleSync` or `ManySync` listener that yields is caught: the generated code closes the suspended thread and warns `[Blink]: Yielded in a Sync call. Event "X" uses a Sync call, consider using an Async call instead.`

> Turning it off removes the check, not the problem. Yielding in a sync listener corrupts the deserialiser's cursor for the rest of that packet.

### `WriteValidations`

```luau
option WriteValidations = true   -- default false
```

Adds type and range checks to the *sending* side. The receiving side is always checked, with or without this option — that is the security boundary, and it cannot be switched off.

> Only builtin primitives are checked (`typeof`, numeric bounds, string and buffer lengths, vector magnitude, `Instance:IsA`). Structs, maps and enums are not.

### `FutureLibrary`

```luau
option FutureLibrary = "ReplicatedStorage.Packages.Future"
```

The require path pasted into the generated module as `local Future = require(<path>)`. Needed before any `function` may use `Yield: Future`; without it the compiler errors with `Cannot use yield type: "Future", without providing a path to the future library.`

> The string is written into the output verbatim, so it is a Luau expression, not a filesystem path.

### `PromiseLibrary`

```luau
option PromiseLibrary = "ReplicatedStorage.Packages.Promise"
```

The same for `Yield: Promise`. evaera's Promise, or a fork of it, is what the docs assume.

### `RemoteScope`

```luau
option RemoteScope = "CROSSWIRE"   -- default: no prefix
```

Prefixes the two RemoteEvents this instance creates: `CROSSWIRE_BLINK_RELIABLE_REMOTE` and `CROSSWIRE_BLINK_UNRELIABLE_REMOTE` instead of `BLINK_…`. Set it in a package so you do not collide with the game that installs you.

> The generated module also registers the scope in `_G._BLINK` and errors at require time if two instances claim the same one.

### `UseColon`

```luau
option UseColon = true   -- parsed, then ignored
```

Accepted by the parser as a boolean option and never read again: `UseColon` appears exactly twice in the whole repository, both in `src/Parser.luau`'s option table and type. It generates nothing.

> Inert in 0.18.8, and in no documentation page. It is listed in the `[E2003]` message, which is the only place you will meet it.

## Primitive types

The sizes below are what the value costs in the packet. Every type may be made optional, and everything with a `Bounds` entry in `src/Settings.luau` accepts a range.

### `u8`

```luau
u8            -- 1 byte, 0 .. 255
u8(0..100)    -- Luau: number
```

The cheapest number. Health, ammo, a percentage, a small index — anything that fits in a byte should be one.

### `u16`

```luau
u16           -- 2 bytes, 0 .. 65,535
-- Luau: number
```

Also the type Blink picks on its own for an unbounded string, buffer or array length prefix.

### `u32`

```luau
u32           -- 4 bytes, 0 .. 4,294,967,295
-- Luau: number
```

The widest unsigned integer. There is no `u64`; a Roblox UserId does not fit here and needs `f64`.

### `i8`

```luau
i8            -- 1 byte, -128 .. 127
-- Luau: number
```

A signed byte. A delta, a small offset, a signed level modifier.

### `i16`

```luau
i16           -- 2 bytes, -32,768 .. 32,767
-- Luau: number
```

The usual quantisation target for positions: `vector<i16>` costs 6 bytes instead of 12.

### `i32`

```luau
i32           -- 4 bytes, -2,147,483,648 .. 2,147,483,647
-- Luau: number
```

The widest signed integer. As with `u32`, there is no 64-bit integer type.

### `f16`

```luau
f16           -- 2 bytes, ±65,504
-- Luau: number
```

A half float, implemented by hand because `buffer` has no native f16 — which makes it slower to read and write than every other number type. Cheap on the wire, not on the CPU.

> The type documentation lists its range as ±2048. `src/Settings.luau` declares ±65,504, which is the real IEEE half maximum; the docs are describing the largest exactly representable integer.

### `f32`

```luau
f32           -- 4 bytes, ±16,777,216
-- Luau: number
```

The default component of `vector` and `CFrame`. The bound quoted is the largest integer a float32 represents exactly, not its maximum magnitude.

### `f64`

```luau
f64           -- 8 bytes, ±9,007,199,254,740,992
-- Luau: number
```

Luau's own number. The widest thing Blink sends, and the only type that holds a Roblox UserId, so `type PlayerId = f64` is the standard spelling.

### `boolean`

```luau
boolean       -- 1 byte
-- Luau: boolean
```

A whole byte per flag. Several flags that travel together are cheaper packed into one `u8` and masked by hand.

### `string`

```luau
string        -- u16 length prefix + the bytes
string(3..20) -- u8 prefix when the max fits one
string(36)    -- exact length: no prefix at all
```

A length prefix followed by the characters. The prefix is the smallest unsigned type that holds your maximum, so bounding a string under 256 characters saves a byte on every send, and an exact length saves two.

> A range on a string bounds its length, never its contents. The receiving side always enforces it.

### `buffer`

```luau
buffer          -- u16 length prefix + the bytes
buffer(..950)   -- Luau: buffer
```

Your own serialised bytes, copied straight through while still riding Blink's batching. Same prefix rule as `string`.

### `vector`

```luau
vector          -- 3 × f32 = 12 bytes
vector<i16>     -- 3 × i16 = 6 bytes
vector(0..1)    -- Luau: Vector3
```

A Vector3. The angle-bracket component picks the encoding; a range bounds the magnitude, checked on arrival. Roblox stores vectors as three f32s, so `vector<f64>` buys nothing.

### `Instance`

```luau
Instance             -- 0 buffer bytes
Instance(BasePart)   -- Luau: BasePart
Instance(Player)?    -- optional costs nothing extra
```

Rides the RemoteEvent's own instance table rather than the buffer, so it costs nothing in the packet. A class in parentheses narrows the Luau type and adds an `:IsA` check; subclasses are accepted.

> If a non-optional Instance arrives as nil — streaming, a client-only object, an instance destroyed in flight — deserialisation errors and the rest of that packet is dropped. Mark it `Instance?` if it might not exist. An optional Instance is the one optional that costs no extra byte: the reader peeks at the instance table instead of writing a presence flag.

### `unknown`

```luau
unknown       -- 0 buffer bytes
-- Luau: any
```

Anything, sent through the remote's instance table using Roblox's own serialisation. That is exactly what Blink exists to avoid, so use it sparingly; for a union, reach for a tagged enum.

> `src/Settings.luau` marks it `AllowsOptional = false`, but that field is never read: `unknown?` compiles and types as `any?`. It already carries nil of its own accord, through a sentinel in the instance table.

### `CFrame`

```luau
CFrame              -- 24 bytes (position + orientation, f32)
CFrame<i16, f16>    -- 12 bytes
-- Luau: CFrame
```

A position and an orientation, written as three components then three Euler angles. Two angle-bracket components pick the encodings.

> The documentation says the two components are "positional and rotational respectively". The source is the other way round: `Types.CFrame` uses `Components[1]` for the rotation and hands the whole list to `Types.vector`, which takes `Components[#Components]` — the last — for the position. An integer type in the *first* slot is what zeroes the rotation out.

### `Color3`

```luau
Color3        -- 3 bytes, one u8 per channel
-- Luau: Color3
```

Sent as `Color3.fromRGB`, so a colour round-trips at 8 bits per channel and anything finer is lost.

> `BrickColor` is a separate primitive: 2 bytes, the palette number as a u16, reconstructed with `BrickColor.new`.

### `DateTime`

```luau
DateTime      -- 8 bytes, the Unix timestamp as f64
-- Luau: DateTime
```

Sent as `UnixTimestamp` and rebuilt with `DateTime.fromUnixTimestamp`, so it carries whole seconds.

> `DateTimeMillis` is the millisecond variant — also 8 bytes, also typed `DateTime` in Luau, but `fromUnixTimestampMillis` on the way back. It is named in the type docs' "Other Roblox Types" list and nowhere else.

## Composite types and modifiers

Everything you build out of the primitives. Each of these is a `Declarations` case in `src/Generator/init.luau`.

### `Ranges`

```luau
u8(0..100)    -- full
u8(0..)       -- half: no upper bound
string(36)    -- exact: min and max are the same
```

A range narrows what a value may be, never how it is stored: `u8(0..100)` is still one byte. On numbers it bounds the value, on strings and buffers the length, on arrays the element count, on vectors the magnitude.

> The reader always asserts a range. The writer only does with `WriteValidations = true`, which is why a bad value from a client is caught and a bad value from your own server code is not. A range on a type that has no bounds — `boolean(0..1)` — parses and is then silently ignored: `AllowsRange` in `src/Settings.luau` is never read.

### `? (optional)`

```luau
type Nickname = string(..20)?
Data: (u8, vector?)
```

Appended to the *whole* type, never to part of it. The writer emits a one-byte presence flag and then the value only if it is there, so an absent optional costs one byte and nothing else.

> `Instance?` is the exception and costs no flag byte: the reader peeks at the instance table rather than writing a flag. A map's keys and values may not be optional; everything else may.

### `[] (array)`

```luau
u16[]         -- unbounded: a u16 count
u16[1..50]    -- bounded
u16[3]        -- exact length: no count at all
```

A list of one type. The count prefix follows the same rule as a string's length prefix, and an exact length removes it entirely.

> An unbounded array defaults to 0..65,535 elements, which is a lot of packet. Bound anything a client can send.

### `struct`

```luau
struct Entity {
	Health: u8(0..100),
	Position: vector,
	Animations: struct { First: u8?, Second: u8 },
}
```

A fixed set of named fields, written in declaration order with no keys on the wire. Structs nest inline, so a sub-struct costs nothing but its fields.

> Field order is part of the format. Reordering fields changes the bytes, so server and client must always be generated from the same file.

### `.. (struct merge)`

```luau
struct Both {
	..Base,
	..Extra,
	Own: u8,
}
```

Copies another struct's fields into this one — a table union in Luau terms. `Both` above types as `{ Id: number, Tag: string, Own: number }`.

### `map`

```luau
map Scores = { [string]: u16 }
map Scores = { [string]: u16 }(1..100)
```

A key-value table with one key type and one value type. A range bounds the number of entries. Neither the key nor the value may be optional — that is `[E3002] Maps cannot have optionals as keys or values` — but a map may hold another map.

> `map`, `struct` and tagged `enum` take generics: `map Pair<K, V> = { [K]: V }` then `map IdToName = Pair<f64, string>`. A generic declaration cannot be `export`ed.

### `set`

```luau
set Flags = {
	FeatureA,
	FeatureB,
}
```

A fixed list of names, each present or not. One bit of meaning per name, and the Luau type is a table of those names to booleans.

> The documentation calls a set "a map of static string keys to `true`"; the generated type is `{FeatureA: boolean, FeatureB: boolean}`, so a missing flag reads as `false`, not `nil`.

### `enum (unit)`

```luau
enum Team = { Red, Blue }
-- Luau: ("Red" | "Blue")
```

A closed set of values, sent as an index rather than the string. Two variants cost one byte, and so do two hundred and fifty-six.

### `enum "Tag" (tagged)`

```luau
enum KillFeed = "Kind" {
	Melee { Killer: f64 },
	Ranged { Killer: f64, Distance: u16 },
}
```

Blink's union. One byte picks the variant and only that variant's fields are written, so the packet costs the tag plus the branch you took. The Luau type is a real discriminated union on the tag field.

> The tag name cannot also be a field inside a variant — that is `[E3005]`.

## Declarations

The keywords that are not type constructors. `struct`, `map`, `set` and `enum` declare at the top level too — they are in the group above.

### `type`

```luau
type Health = u8(0..100)
export type Health = u8(0..100)
```

Names a type so events, structs and other types can refer to it. Every named type becomes an `export type` in the generated modules, prefixed with its scope if it has one.

> `export` on a declaration means something else entirely — see the generated API's `Read`/`Write` entry. It is refused on `event`, `function` and `scope`, and on anything generic.

### `event`

```luau
event Move {
	From: Client,        -- Client | Server
	Type: Unreliable,    -- Reliable | Unreliable
	Call: Polling,       -- SingleSync | SingleAsync | ManySync | ManyAsync | Polling
	Data: vector<i16>,   -- optional; a tuple in parentheses for several values
}
```

One message. `From` names the sender, so the other side is the one that gets a listener. `Type: Unreliable` maps to an UnreliableRemoteEvent — unordered, droppable, and capped by Roblox at 1000 bytes a fire. `Call` decides the receiving API.

> That 1000-byte cap is Roblox's and the compiler never checks it — budget unreliable payloads yourself. A fifth field, `Poll: true`, is in the parser and in no documentation page; it overrides `Call` with `Polling`. `StepReplication` is reserved and cannot name an event.

### `function`

```luau
function RequestLoadout {
	Yield: Coroutine,   -- Coroutine | Future | Promise
	Data: u8,           -- optional
	Return: Loadout,    -- optional
}
```

A client question and a server answer, always on the reliable channel and always client-to-server. `Yield` picks how the caller waits; `Future` and `Promise` require their library option first.

> At most 256 invocations may be in flight at once; the 257th errors on the client instead of silently dropping. A handler that errors is caught by the generated `pcall` and reaches the caller as "There was an exception while processing …".

### `scope`

```luau
scope Match {
	type Round = u8
	event Started { From: Server, Type: Reliable, Call: SingleAsync, Data: Round }
}
```

Groups declarations inside one file. The generated module nests them — `Blink.Match.Started.FireAll(…)` — and the Luau types are joined with an underscore, `Blink.Match_Round`. A scope sees everything declared in its parents.

> Scopes nest as deep as you like — `scope A { scope B { type X = u8 } }` types as `A_B_X` — even though `src/Modules/Error.luau` reserves an `AnalyzeNestedScope` code (`[E3006]`) that nothing in 0.18.8 ever raises.

### `import`

```luau
import "./crosswire"
import "./crosswire" as Shared

type Health = Shared.Health
```

Pulls another definition file in as a scope, named after the file or after the alias you give. Everything it declares comes with it, prefixed, whether this file names it or not.

> The alias is a bare identifier, not a string: `as "Shared"` is `[E2002]`, though the imports page writes it that way. A path that resolves to nothing is `[E3014] Unknown require`.

## The generated API

What the two modules actually return. Signatures below are Pascal casing and the server's point of view unless stated; `Casing` renames all of them.

### `Fire`

```luau
-- server (From: Server)
Blink.MyEvent.Fire(Player: Player, Value: T): ()
-- client (From: Client)
Blink.MyEvent.Fire(Value: T): ()
```

Sends one message. The server names its recipient; the client does not. On a reliable event nothing leaves the machine here — the payload is appended to that frame's buffer and flushed by `StepReplication`. An unreliable event is sent on the spot, in its own packet.

### `FireAll`

```luau
Blink.MyEvent.FireAll(Value: T): ()
```

Server only. Serialises once and copies the finished bytes into every player's buffer, so the cost per extra player is a memcpy rather than another encode.

### `FireList`

```luau
Blink.MyEvent.FireList(List: {Player}, Value: T): ()
```

The same single encode, delivered to the players you name. The right call for team chat, party UI, or anything an area-of-interest filter produced.

### `FireExcept`

```luau
Blink.MyEvent.FireExcept(Except: Player, Value: T): ()
```

Every player but one. The usual shape for echoing a player's own action to everyone else, when that player already predicted it locally.

### `On`

```luau
-- event, on the receiving side
local Disconnect = Blink.MyEvent.On(Listener: (Player: Player, Value: T) -> ()): () -> ()
-- function, on the server
Blink.MyFunction.On(Listener: (Player, T) -> (R)): ()
```

Registers the listener. On an event it returns a disconnect function; the `Player` first argument appears only on the server. `SingleSync` and `SingleAsync` keep exactly one listener, so a second `On` replaces the first; `ManySync` and `ManyAsync` keep a list.

> `*Sync` listeners are called inline and must not yield; `*Async` listeners are `task.spawn`ed. A function's `On` returns nothing — it has one handler and no disconnect. Reliable messages that arrive before any `On` are queued and replayed when one registers, with a warning naming the event once the queue passes 256. Unreliable ones are not queued: with no listener, they are gone.

### `Invoke`

```luau
-- Yield: Coroutine
Blink.MyFunction.Invoke(Value: T): (R)
-- Yield: Future
Blink.MyFunction.Invoke(Value: T): Future.Future<boolean, R>
-- Yield: Promise
Blink.MyFunction.Invoke(Value: T)
```

Client only. `Coroutine` yields the calling thread and returns the answer; `Future` wraps the call in `Future.Try`; `Promise` returns `Promise.new(…)` and cancelling it drops the pending call.

> The Promise form is generated with no return annotation at all, so a typed editor gives you nothing back — annotate the call site yourself if you use it.

### `Iter`

```luau
for Index, Player, Value in Blink.MyEvent.Iter() do
	-- ...
end
```

The polling API, from `Call: Polling` or `option UsePolling = true`. Each call drains what has arrived since the last one, which is what a scheduled system wants: no callbacks, no ordering surprises, one place in the frame where network input enters the world.

> `Next()` is generated beside it and still works, under a literal `--- @deprecated v0.14.1` comment pointing at `Iter`. Use `Iter`.

### `StepReplication`

```luau
Blink.StepReplication(): ()
```

Flushes everything buffered since the last call. It is on both modules and at the top level, never inside a scope, and it is a reserved name you cannot use for your own declaration.

> Exported whether or not `ManualReplication` is set — that option only decides whether the module also connects it to `RunService.Heartbeat`.

### `export (Read / Write)`

```luau
export struct Snapshot { Position: vector<i16>, Health: u8 }

Blink.Snapshot.Write(Value: Snapshot): buffer
Blink.Snapshot.Read(Buffer: buffer): Snapshot
```

`export` in front of a type declaration hands you its serialiser as a standalone pair on both modules — a buffer in, a value out. This is how a snapshot gets packed into an ECS replication payload without going through an event at all.

> Written `export struct X { … }`, with no `=`; the type docs show one and it does not parse. The docs also say exports do not support `Instance` or `unknown` — 0.18.8's parser accepts both anyway, so the restriction is yours to keep. Generic types cannot be exported.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Rokit](https://github.com/rojo-rbx/rokit) | The toolchain manager that installs the compiler | 1.2.0 | `rokit add 1Axen/blink` |
| [pesde](https://github.com/pesde-pkg/pesde) | The other package manager, with an official Blink publish | 0.7.3 | `pesde add --dev 1axen/blink` |
| [Blink Editor](https://create.roblox.com/store/asset/77231976488966/Blink-Editor) | Studio plugin: write, analyse and generate without leaving Studio | Creator Store | `asset 77231976488966` |
| [Zap](https://zap.redblox.dev/) | The other networking IDL for Roblox | 0.6.29 | `rokit add red-blox/zap` |
| [ByteNet](https://github.com/ffrostfall/ByteNet) | Runtime packet library — the same goal without a compiler | 0.4.6 | `ffrostflame/bytenet@0.4.6` |
| [jecs](https://github.com/Ukendio/jecs) | The ECS a polling network layer feeds | 0.11.0 | `ukendio/jecs@0.11.0` |
| [Future](https://github.com/red-blox/Util/tree/main/Libraries/Future) | redblox's future library, for `Yield: Future` | — | `option FutureLibrary = "ReplicatedStorage.Packages.Future"` |
| [Promise](https://github.com/evaera/roblox-lua-promise) | evaera's promise library, for `Yield: Promise` | — | `option PromiseLibrary = "ReplicatedStorage.Packages.Promise"` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`export struct X = { … }`.** The Exports section of the type documentation shows an `=`. The parser rejects it — `[E2002] … Expected "OpenBraces", found "Assign"`. Write `export struct X { … }`, with no `=`.
- **`import "./external" as "Common"`.** The imports page quotes the alias. A quoted alias does not parse: `[E2002] … Expected one of "As", "Import", "Boolean", "Keyword", "Identifier", found "String"`. The alias is a bare identifier — `as Common`.
- **`Next()` for polling.** Still generated, and still works, but the generator writes a `--- @deprecated v0.14.1` comment immediately above it, pointing at `Iter`. Use `Iter()`.
- **"`ManualReplication` exposes a `StepReplication` function instead."** It is exported either way. The option only decides whether the generated module also connects it to `RunService.Heartbeat`.
- **`f16` ranging ±2048.** That is the type documentation's figure. `src/Settings.luau` declares the bounds as ±65,504, the real IEEE half maximum; the docs are quoting the largest exactly representable integer.
- **A set's Luau type as `{Name: true}`.** The docs describe a set as "a map of static string keys to `true`". The generated type is `{FeatureA: boolean, FeatureB: boolean}`, so an absent flag reads `false`.
- **`u64` and `i64`.** They do not exist. Nine number types, and nothing wider than 32 bits except the floats — a UserId goes in an `f64`.
- **`Vector3` as a type name.** In Blink the type is `vector`, lower case, with `vector<i16>` for the encoding. `Vector3` is only what it becomes in Luau.
- **`option UseColon` and the per-event `Poll: true` field.** Both are in `src/Parser.luau` and in no documentation page. `Poll: true` overrides `Call` with `Polling`; `UseColon` is parsed and then never read, and generates nothing.
- **Anything written against `v1.0.0-pre.*`.** That is a different compiler on the `rewrite` branch — a new tree built on Lute rather than Lune. Nothing written for it applies to `v0.18.8`, and nothing here applies to it.
