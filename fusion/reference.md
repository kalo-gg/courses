# Fusion reference

*Fusion 0.3, released August 30, 2024. Checked on September 7, 2026.*

---

## Memory: scopes and cleanup

Fusion 0.3's headline idea. A scope is an array of destructors; everything you create goes in one, and the scope is destroyed as a unit.

### `scoped`

```luau
Fusion.scoped(...: Constructors) -> Scope<Constructors>
```

Creates a scope whose methods are the tables you pass in, so `scoped(Fusion)` gives a scope you can call every Fusion constructor on: `scope:Value(...)`, `scope:New(...)`.

> `scoped(Fusion, myUtilities)` merges several tables, which is how projects add their own components to the same call syntax. A key defined in two of them is a fatal `Multiple definitions for '…' found while merging.`

### `doCleanup`

```luau
Fusion.doCleanup(task: Task)
```

Destroys everything in a scope. Per the source: an Instance is destroyed, an RBXScriptConnection is disconnected, a function is run, a table with `destroy` or `Destroy` has it called, and an array has each item cleaned up in turn — walked backwards (`for index = #task, 1, -1`), so objects are destroyed in reverse creation order.

> Reverse order matters: an object is always destroyed before whatever it was built from.

### `deriveScope`

```luau
Fusion.deriveScope<T>(existing: Scope<T>, ...) -> Scope<T & A & B & …>
```

Creates a new, empty scope with the same methods as an existing one, without naming the constructor tables again.

### `innerScope`

```luau
Fusion.innerScope<T>(existing: Scope<T>, ...) -> Scope<T & A & B & …>
```

In the source's own words, derives a new scope "that's destroyed exactly once, whether by the user or by the scope that it's inside of". It inserts itself into the existing scope, so destroying the outer one destroys it too, and destroying it first removes it from the outer scope rather than leaving a dead entry behind.

> The usual tool for UI that comes and goes while its parent stays.

### `Scope`

```luau
export type Scope<Constructors = any> = {Task} & Constructors
```

The type itself: an array of tasks, plus whatever constructors were merged in. A scope really is just a list of things to destroy later.

### `Task`

```luau
export type Task =
  Instance | RBXScriptConnection | () -> ()
  | {destroy: (unknown) -> ()} | {Destroy: (unknown) -> ()}
  | {Task}
```

Everything `doCleanup` knows how to destroy. Anything matching this can be put in a scope, including your own objects.

## State objects

Values that other things can watch. Every constructor takes the scope first.

### `Value`

```luau
Fusion.Value<T>(scope: Scope<unknown>, initialValue: T) -> Value<T, any>
value:set(newValue: S) -> S
```

A state object you write to. `:set` stores a new value, wakes anything watching it, and **returns the value you passed** — which is what makes the `:set()` reference idiom work.

> `:set` skips the update when the new value is *similar* to the old: `==` for primitives, userdata, frozen tables and tables with metatables — and every other table counts as changed. There is no force argument in 0.3.

### `Computed`

```luau
Fusion.Computed<T, S>(scope: S & Scope<unknown>, callback: (Use, S) -> T) -> Computed<T>
```

A value derived from other state. The callback receives `use`, which reads a state object and records the dependency, and the scope, so objects created inside are cleaned up with it. When a dependency changes the computed "throws everything away and recalculates", though Fusion may postpone or cancel that work while nothing is reading the result.

> Two rules follow from that: the callback must be immediate (never yield), and it must not have side effects — the docs single out playing a sound as the thing not to do here, because the recalculation might never run.

### `Observer`

```luau
Fusion.Observer(scope: Scope<unknown>, watching: unknown) -> Observer
observer:onChange(callback: () -> ()) -> (() -> ())
observer:onBind(callback: () -> ()) -> (() -> ())
```

Runs a callback as a side effect when what it watches changes. `onChange` fires on later changes; `onBind` also runs once immediately. Both return a function that disconnects that one listener.

> An observer is how you reach the world outside Fusion. Its callback always runs, where a `Computed`'s work can be postponed while nothing reads the result — so side effects belong here, not there.

### `peek`

```luau
Fusion.peek<T>(target: UsedAs<T>) -> T
```

Reads a state object's current value without recording a dependency. For when you want the number, not a subscription.

### `Use`

```luau
export type Use = <T>(target: UsedAs<T>) -> T
```

The reader passed into `Computed` and friends. Calling it both returns the value and marks it as a dependency.

### `UsedAs`

```luau
export type UsedAs<T> = StateObject<T> | T
```

Fusion's way of saying "a T, or something that produces a T". Properties and animation goals take `UsedAs`, which is why you can pass either a constant or a state object.

### `StateObject`

```luau
export type StateObject<T> = GraphObject & {type: "State", kind: string, …}
```

The type every readable Fusion object satisfies: `Value`, `Computed`, `Observer`'s subject, the `For` family, `Tween` and `Spring` are all state objects, which is why they are interchangeable anywhere `UsedAs` appears.

> This is what makes an animation composable: a `Spring` can be a property's value, a `Computed`'s dependency or another animation's goal.

## Tables and lists

One object per member of the family, and each differs only in which half of the table it touches.

### `For`

```luau
export type For<KO, VO> = StateObject<{[KO]: VO}> & {kind: "For"}
```

What all three constructors return: a state object holding the processed table. `peek` it for a plain table, or drop it into `[Children]`.

> It is a state object, so a `Computed` can `use` it and a property can be bound to it.

### `ForValues`

```luau
Fusion.ForValues<K, VI, VO, S>(
  scope: S & Scope<unknown>,
  inputTable: UsedAs<{[K]: VI}>,
  processor: (Use, S, value: VI) -> VO
) -> For<K, VO>
```

In the source's words, "processes values and preserves keys". The processor is handed `use` and a scope of its own, just as a `Computed` is, plus the value being processed — so anything it creates is cleaned up when that value leaves the table.

> It does not rebuild the output from scratch: it "will try and reuse as much as possible", following a value that moves position rather than recomputing it. Each value is only reused once, so a table with duplicates recomputes the repeats.

### `ForKeys`

```luau
Fusion.ForKeys<KI, KO, V, S>(
  scope: S & Scope<unknown>,
  inputTable: UsedAs<{[KI]: V}>,
  processor: (Use, S, key: KI) -> KO
) -> For<KO, V>
```

"Processes keys and preserves values" — the mirror image of `ForValues`, with the same processor shape.

> Fusion's own `scopeMissing` error message prints the processor arguments in the order `(scope, use, …)`; `Types.luau` is the authority and the real order is `(use, scope, …)`, which is what the constructors pass.

### `ForPairs`

```luau
Fusion.ForPairs<KI, KO, VI, VO, S>(
  scope: S & Scope<unknown>,
  inputTable: UsedAs<{[KI]: VI}>,
  processor: (Use, S, key: KI, value: VI) -> (KO, VO)
) -> For<KO, VO>
```

"Processes keys and values in pairs", for when the new key depends on the value or the other way round.

> Fusion's own `scopeMissing` error message prints the processor arguments in the order `(scope, use, …)`; `Types.luau` is the authority and the real order is `(use, scope, …)`, which is what the constructors pass.

## Instances

Constructors are curried: the scope and class first, then the property table.

### `New`

```luau
Fusion.New(scope: Scope<unknown>, className: string)
  -> (propertyTable: PropertyTable) -> Instance
```

Creates an instance and puts it in the scope. Written `scope:New "TextLabel" { … }`, which is the curry in action.

### `Hydrate`

```luau
Fusion.Hydrate(scope: Scope<unknown>, target: Instance)
  -> (propertyTable: PropertyTable) -> Instance
```

Takes an instance that already exists — one built in Studio, say — and binds Fusion state to its properties.

> It adopts the target with the same `table.insert(scope, target)` that `New` uses, so `doCleanup` destroys it. Hydrate a `:Clone()` when the original must survive.

### `Children`

```luau
[Fusion.Children] = …
```

The special key that parents instances, arrays of instances, or state objects containing them.

> A state object holding `nil` means "no child here"; the key only ever writes `Parent`, so destruction is the owning scope's job.

### `OnEvent`

```luau
[Fusion.OnEvent "Activated"] = function(...) … end
```

Connects an event on the instance being created; the connection belongs to the scope.

### `OnChange`

```luau
[Fusion.OnChange "Text"] = function(newValue) … end
```

Runs when a Roblox property changes, including changes Fusion did not make.

### `Out`

```luau
[Fusion.Out "Text"] = someValue
```

**Instance to state, one way:** writes the instance's property back into a `Value` as it changes.

> Setting the `Value` does **not** move the property back. For real two-way binding, bind the property as well: `Text = typed, [Out "Text"] = typed`.

### `Attribute`

```luau
[Fusion.Attribute "Name"] = …
```

Sets an attribute from a constant or state object.

### `AttributeChange`

```luau
[Fusion.AttributeChange "Name"] = function(newValue) … end
```

Runs when one attribute changes.

### `AttributeOut`

```luau
[Fusion.AttributeOut "Name"] = someValue
```

Writes an attribute back into a `Value`.

> It must be given a `Value`; anything else is the fatal `invalidAttributeOutType`.

### `Child`

```luau
export type Child = Instance | StateObject<Child> | {[unknown]: Child}
Fusion.Child(x: {Child}) -> Child
```

What `[Children]` accepts: an instance, something producing one, or any nesting of those. `Fusion.Child` is also an exported function — a cast helper for type checking only: it returns its argument, and exists to stop Luau inferring a narrower array type for a `[Children]` list.

> Runtime behavior is unchanged with or without the call.

### `PropertyTable`

```luau
export type PropertyTable = {[string | SpecialKey]: unknown}
```

The second half of `New` and `Hydrate`: property names, and special keys like `[Children]` or `[OnEvent "Activated"]`, in one table.

### `SpecialKey`

```luau
export type SpecialKey = {
  type: "SpecialKey",
  kind: string,
  stage: "self" | "descendants" | "ancestor" | "observer",
  apply: (…) -> ()
}
```

What `Children`, `OnEvent`, `OnChange`, `Out`, `Attribute`, `AttributeChange` and `AttributeOut` all are. `stage` is why `[Children]` is applied after ordinary properties.

> The set is closed in 0.3; there is no supported way to write your own.

## Animation

Animations are state objects too: they take a goal that is itself UsedAs, so they follow other state.

### `Tween`

```luau
Fusion.Tween<T>(scope: Scope<unknown>, goalState: UsedAs<T>, tweenInfo: UsedAs<TweenInfo>?) -> Tween<T>
```

Follows its goal along a `TweenInfo` curve. Change the goal and it animates there.

> The `TweenInfo` argument is itself `UsedAs`, so an easing curve can change with your state.

### `Spring`

```luau
Fusion.Spring<T>(scope: Scope<unknown>, goalState: UsedAs<T>, speed: UsedAs<number>?, damping: UsedAs<number>?) -> Spring<T>
spring:setPosition(newPosition: T)
spring:setVelocity(newVelocity: T)
spring:addVelocity(deltaVelocity: T)
```

Physical motion towards its goal, with speed and damping that can themselves be state. `setPosition`, `setVelocity` and `addVelocity` nudge the simulation directly.

> None of the three moves the goal, and none of them works on a spring whose goal is a plain constant: `Spring.luau` only builds a stopwatch when the goal is a state object, so a constant-goal spring never simulates and `setPosition` raises `springTypeMismatch`. Speed and damping are not validated at all — see the `messages` group.

### `Animatable`

```luau
export type Animatable =
  number | CFrame | Color3 | ColorSequenceKeypoint | DateTime | NumberRange
  | NumberSequenceKeypoint | PhysicalProperties | Ray | Rect | Region3
  | Region3int16 | UDim | UDim2 | Vector2 | Vector2int16 | Vector3 | Vector3int16
```

Every type `Tween` and `Spring` can animate: the ones Fusion can express as a vector of numbers.

> In the Kalo sandbox a `Spring` covers all of these, while a `Tween` runs over `number` and `UDim` only — the stand-in value types have no `Lerp`. That is a sandbox limit, not a Fusion one.

## What Fusion tells you

Fusion names its own failure modes. Warnings and non-fatal errors are printed and execution continues; the fatal ones stop the thread. Learn these eight and most lifetime bugs read themselves.

### `possiblyOutlives`

```luau
(warning) "%s will be destroyed before %s; %s. To fix this, review the order they're created in, and what scopes they belong to. See discussion #292 on GitHub for advice."
```

Something depends on an object with a shorter life. Move one of them into the other's scope.

> The third slot is either "they're in the same scope, but the latter is destroyed too quickly" or "the latter is in a different scope that gets destroyed too quickly". `formatError` wraps every message, so what reaches the output is `[Fusion] <text>` then an indented `ID: possiblyOutlives` and a `---- Stack trace ----` block. In the browser sandbox the prefix appears twice, because the shim's `logWarn` prepends its own to the one `formatError` already wrote; a real game prints it once.

### `useAfterDestroy`

```luau
(fatal) "%s is no longer valid - it was destroyed before %s. See discussion #292 on GitHub for advice."
```

`possiblyOutlives` one step later: the read actually happened after the destruction.

> Wrapped the same way as every other message, so the printed line ends with an indented `ID: useAfterDestroy`.

### `poisonedScope`

```luau
(fatal) "Attempted to use a scope after it's been destroyed; %s"
```

`doCleanup` clears a scope and replaces its metatable, so every later index or assignment errors. A scope is single-use.

> `innerScope` is the supported way to have a lifetime you can end early and then replace.

### `destroyedTwice`

```luau
(fatal) "`doCleanup()` was given something that it is already cleaning up. Unclear how to proceed."
```

One task owned by two scopes, or cleaned up by hand and then by its scope.

### `callbackError`

```luau
(fatal or not, depending on who raises it) "Error in callback:\nERROR_MESSAGE"
```

A `Computed` or a `For` processor threw: both wrap the call in `xpcall` and report through `logErrorNonFatal`, so Fusion prints this and keeps the previous value. `Contextual:during` raises the same id through `logError`, which is fatal.

> An `Observer` callback never raises it at all — observers pass the callback straight to `External.doTaskImmediate` with nothing around it, so an error inside one propagates out of the `:set` that triggered it and kills the script.

### `forKeyCollision`

```luau
(non-fatal) "The key '%s' was returned multiple times simultaneously, which is not allowed in `For` objects."
```

`ForKeys` or `ForPairs` produced a duplicate output key; the duplicate is dropped.

### `invalidOutProperty / invalidOutType`

```luau
(fatal) "The %s class doesn't have a property called '%s'." / "[Out] properties must be given Value objects."
```

The two ways an `[Out]` key goes wrong: a property that does not exist, or a target that is not a `Value`. `[AttributeOut]` has its own `invalidAttributeOutType`, also fatal.

> All three call `External.logError`, which calls `error()`, so the script stops on the line that built the instance rather than carrying on with a dead binding.

### `stateGetWasRemoved`

```luau
"`StateObject:get()` has been replaced by `use()` and `peek()` - see discussion #217 on GitHub."
```

The message a pasted 0.2 snippet produces. It is the fastest way to date a tutorial.

## General

### `version`

```luau
Fusion.version -> {major: number, minor: number, isRelease: boolean}
```

The running version. On the `v0.3-beta` tag this is `{major = 0, minor = 3, isRelease = true}`; on `main` it currently reads minor 4 with `isRelease = false`.

> Worth checking when a tutorial's API does not match yours.

### `Contextual`

```luau
local ctx = Fusion.Contextual(defaultValue)
ctx:now() -> T
ctx:is(newValue):during(callback, ...)
```

"Time-based contextual values, to allow for transparently passing values down the call stack." `:now()` reads the current value; `:is(v):during(fn)` runs `fn` with the value temporarily set, restoring the old one afterwards even if `fn` errors.

> Reach for it instead of threading a theme or a player through every component's props. It is time-based, so a callback that runs later — an event handler, a `For` processor re-running — reads the default, not what was set during the build.

### `cleanup`

```luau
Fusion.cleanup(value: Task)  -- deprecated
```

The old name for `doCleanup`, kept so 0.2 code still runs. `src/Memory/legacyCleanup.luau` logs the warning `cleanupWasRenamed` and then calls `doCleanup` for you.

> If you see this warning in the output window, a snippet you pasted is older than 0.3.

### `Safe`

```luau
Fusion.Safe<Success, Fail>(callbacks: {
  try: () -> Success,
  fallback: (err: unknown) -> Fail
}) -> Success | Fail
```

"A variant of xpcall() designed for inline usage, letting you define fallback values based on caught errors." Returns whichever of the two callbacks produced a value.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Fusion](https://github.com/dphfox/Fusion) | The library itself | 0.3.0 | `Fusion = "elttob/fusion@0.3.0"` |
| [Wally](https://wally.run) | Package manager for Roblox | 0.3.2 | `wally install` |
| [Rojo](https://rojo.space) | Syncs a source tree into Studio | 7.7.0 | `rojo serve` |
| [Luau LSP](https://github.com/JohnnyMorganz/luau-lsp) | Editor support: types and autocomplete for Fusion's API | 1.69.0 | `JohnnyMorganz.luau-lsp` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **Constructors without a scope.** Almost every tutorial written before August 2024 shows `Fusion.Value(0)` or `New "TextLabel" { … }`. In 0.3 the scope always comes first: `scope:Value(0)`, `scope:New "TextLabel" { … }`.
- **`:get()` on state objects.** 0.3 reads state through the `use` function a `Computed` is handed, or through `peek` when you deliberately do not want a dependency.
- **`Computed(function() return thing:get() * 2 end)`.** The 0.3 callback receives `use` and the scope: `scope:Computed(function(use) return use(thing) * 2 end)`.
- **A `cleanupWasRenamed` warning.** `Fusion.cleanup` is the 0.2 name; 0.3 kept it as a deprecated shim that warns and forwards to `doCleanup`.
- **Anything mentioning `Tag` or `insert`.** Those are on `main` for 0.4, which has not been released; they do not exist in 0.3.
- **`[Ref] = someValue`.** `Fusion.Ref` was removed in 0.3 and is not exported at all — only a dead `invalidRefType` message survives. The replacement is that `:set()` **returns its argument**, so you can capture a reference inline: `selectionTarget:set(scope:New "Part" { … })` inside a `[Children]` list both builds the part and hands it to the value.
- **`:set(value, true)`.** The `force` parameter is declared in `Types.luau` and implemented nowhere, in 0.3 or on `main`. If a re-set is not firing, the value is *similar* — a frozen table or one with a metatable — and the fix is a new table, not a flag.
- **A tutorial that binds a property to a bare function.** 0.2 accepted `Text = function() … end` in some shapes. In 0.3 a property takes `UsedAs<T>` — a constant or a state object — and a function is neither. Wrap the logic in `scope:Computed(function(use) … end)`.
- **`ForValues(inputTable, function(scope, use, value) … end)`.** Fusion's own `scopeMissing` error message prints the processor arguments in that order, and it is wrong: `Types.luau` and the constructors both pass `(use, scope, value)`. Snippets copied out of an error message have the first two swapped.
