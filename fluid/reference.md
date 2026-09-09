# Fluid reference

*Fluid 0.7.0-rc.7, released commit a93f33a, 25 July 2026. Checked on September 7, 2026.*

---

## The reactive graph

Sources hold values, derives compute from them, effects react. Every name is snake_case.

### `source`

```luau
fluid.source<T>(initial_value: T) -> Source<T>
src() -> T    -- read
src(value)    -- write
```

A value you can read and write. Calling it with no arguments reads and, inside a reactive scope, subscribes; calling it with one writes.

### `derive`

```luau
fluid.derive<T>(compute: (previous: T) -> T) -> Readable<T>
```

A cached value computed from other sources. Recomputes when something it read changes. The callback is handed its own previous result — `nil` on the first run — so a derive can compare against what it last produced.

> Must be created inside a stable scope. At the top level it errors with `cannot call derive without a parent scope`; called from inside an `effect`, a `derive` or a reactive property it errors with `… while the parent scope is reactive`, and the fix named in the message is `untrack`.

### `effect`

```luau
fluid.effect<T>(callback: (previous: T) -> T, initial_value: T?)
```

Runs a callback now and again whenever a source it read changes. Where side effects belong. Like `derive`, the callback receives its own previous return value; `initial_value` seeds the first run.

> Also needs a stable scope, and like every reactive callback it may not yield — no `task.wait`, no `:WaitForChild`, no `:InvokeServer`.

### `root`

```luau
fluid.root<T...>(fn: (destroy: Cleanup) -> T...) -> (Cleanup, T...)
```

Creates a stable scope and runs `fn` inside it. Returns the destructor first, then whatever `fn` returned.

> Almost every Fluid program starts here. The list of things that refuse to run outside a stable scope is long: `derive`, `effect`, `deferred`, `async`, `interval`, `for_values`, `for_keys`, `switch_delay` (and `show_delay`), `tags`, `lerp`, `spring`, and `create` whenever any property in its table is a function.

### `untrack`

```luau
fluid.untrack<T>(source: () -> T) -> T
```

Reads without subscribing — Fluid's equivalent of reading a value you do not want to depend on.

### `cleanup`

```luau
fluid.cleanup(value: unknown)
```

Registers something to be destroyed when the surrounding scope is: a function is called, an instance is destroyed. Instances built by `create` are torn down only this way.

### `deferred`

```luau
fluid.deferred<T>(callback: (previous: T) -> T, initial_value: T?)
```

An `effect` that runs in the deferred phase instead of eagerly. Its node is queued into `deferred_nodes` and evaluated when the scheduler flushes them, so several writes in one frame cost one run rather than one run each.

> `deferred` is the only thing the user API puts in that queue directly. Fluid also defers reactive children, reactive `tags`, and the input nodes of `lerp` and `spring`. `async` nodes are **not** deferred — they are invalidated eagerly.

### `read`

```luau
fluid.read<T>(value: T) -> CastIntoValue<T>
```

Unwraps "a value or a function returning one": `if type(value) == "function" then value() else value`. A **one-shot** read, with no subscription.

> Used internally by `for_values`, `for_keys`, `show`, `switch`, `spring` and `lerp` — **not** by `bind` or `create`, whose function-valued properties become live reactive nodes instead.

### `async`

```luau
fluid.async<T>(callback: (set: (value: T) -> ()) -> T)
  -> (() -> T, () -> State)

type State = "busy" | "ok"
```

A reactive node whose value arrives later. Returns a reader for the value and a reader for the state, which is only ever `"busy"` or `"ok"` — there is no failure state. If the callback errors, Fluid logs a warning and the state still becomes `"ok"`, so handle failure inside the callback if you need to show it.

> `set(value)` pushes a value out without returning, so a long callback can publish partial results and keep working; the final `return` is `set` one last time. The node is invalidated eagerly, like an effect — what runs on its own thread, and so may yield, is the callback. Fluid's README lists async nodes among the features "where Fluid has an advantage"; the comparison to Vide and Fusion is the library's claim, not this reference's finding.

### `interval`

```luau
fluid.interval<T>(
  func: (dt: number) -> T,
  hz: number?,
  offset: number?
) -> Readable<T>
```

Runs a callback at a rate in **hertz**, driven by Fluid's scheduler rather than by your own loop, and returns a readable holding the callback's latest return value — the README calls it a "utility to poll values".

> It calls `func` once immediately at creation with `dt = 0`, before any stepping; later calls receive the elapsed time since the last one. The gate is `elapsed <= 1 / hz`, so a gated call needs strictly more than the period — at 60 fps and 2 Hz it fires on frames 31 and 62, not 30 and 60. Count the free call at creation when the numbers matter. `offset` shifts the first gated call. The callback may not yield.

## Instances

Fluid takes properties and children in one mixed table.

### `create`

```luau
fluid.create<ClassName>(class_name)
  -> (props: PropertiesFor<ClassName>) -> ClassName
```

Curried: the class name, then a table holding properties by key and children in the array part together. It returns the concrete class, not a bare `Instance` — which is what the currying buys you. A **plain value** is assigned once; a **function** becomes a live binding; a function on a signal-valued property (`Activated`, `MouseButton1Click`) is connected, and disconnected when the scope ends.

> `PropertiesFor` is generated only for the ~85 classes listed in `create.luau`; any other class name falls back to bare `Instance` typing. Two of those entries are misspelled upstream — `ImageHandLeAdornment` and `SphereHandLeAdornment`, with a capital L — so those two silently lose their typing. A **non-function** value assigned to a signal-valued property is silently ignored.

### `mount`

```luau
fluid.mount(fn: () -> { [any]: any }, inst: Instance) -> () -> ()
```

Runs a component inside a fresh `root` and binds what it builds onto an existing instance. Returns that root's destructor — which stops the graph, but does not destroy the instances it parented.

### `bind`

```luau
fluid.bind(inst: Instance, props: { [any]: any }) -> Instance
```

`create`'s applier, exposed. The instance comes **first** and the same mixed table second, and it returns the instance — `create(class)(props)` is literally `bind(Instance.new(class), props)`. Use it on an instance you did not build: a Studio template, a cloned prefab.

> Get the order wrong and nothing warns you. `fluid.bind("Name", src)` does not error: Luau treats the source as a generic-for iterator, calls it with `nil`, and in Fluid a call with an argument is a **write** — so the source is blanked and the loop ends.

### `changed`

```luau
fluid.changed(property: string, fn: (value: any) -> ()) -> Action
```

Calls `fn` with the property's **current** value immediately, then again on every change — the way to read the instance back. It is an action, so it goes in the array part of a `create` table.

> Actions run while the mixed table is being iterated, before the hash-keyed properties are applied and before the instance is parented, so the first call usually sees the class default (often `nil`) rather than the value you wrote in the same table.

### `action`

```luau
fluid.action(callback: (inst: Instance) -> ()) -> Action
fluid.is_action(value) -> boolean
```

A function that runs against the instance, for anything the property table cannot express. Goes in the array part; `is_action` is how Fluid — and your own helpers — tell an action from a child.

> "Once the instance exists" is not "once it is finished": an action runs before the rest of the table's properties are set and before the instance has a `Parent`. Anything an action `:Connect`s is yours to disconnect — unlike an `Activated = fn` property, which Fluid disconnects with the scope.

### `tags`

```luau
fluid.tags(tags: { string | () -> string? }) -> Action
```

An action that adds CollectionService tags and removes them when the scope is destroyed. A function entry becomes a deferred reactive node, so a tag can follow state.

## Control flow and tables

### `show`

```luau
fluid.show<T, U>(condition: () -> any, truthy: UsedAs<T>, falsy: UsedAs<U>?) -> () -> T | U
```

Swaps between two things as a condition changes. With one branch it returns nil when false.

> Both branches are read through `fluid.read`, so each may be a value or a function returning one, and both are read untracked. `show` is a `derive` underneath, so it needs a stable scope.

### `switch`

```luau
fluid.switch<T, U>(condition: () -> T)
  -> (options: { [T]: UsedAs<U> }) -> () -> U?
```

Curried: the condition, then a table of cases. Picks the branch matching the current value; a value with no case reads nil rather than erroring.

> Also a `derive` underneath, so it also needs a stable scope.

### `show_delay`

```luau
fluid.show_delay<T, U>(
  condition: () -> any,
  truthy: (active: Readable<boolean>) -> (T, number?),
  falsy: ((active: Readable<boolean>) -> (U, number?))?
) -> () -> { T | U }
```

Two-branch swap that holds the outgoing branch for the delay it asks for. It is `switch_delay` with `not not condition()` and the keys `true` and `false`.

> Passing plain values, as `show` and `switch` accept, does not error — it silently produces nonsense. The array is built from a hash table, so its order is not guaranteed; iterate it rather than indexing `[1]`.

### `switch_delay`

```luau
fluid.switch_delay<K, R>(condition: () -> K)
  -> (options: { [K]: (active: Readable<boolean>) -> (R, number?) })
  -> () -> { R }
```

Many-branch version. Every branch is a function taking `active` and returning `(value, delay?)`; the result is an array of every live branch — one normally, two mid-transition.

> Branches are keyed by the *function*, so two cases pointing at the same function share one branch and one scope. A key with no case simply produces no incoming branch: the array empties once the outgoing one has had its delay.

### `for_values`

```luau
fluid.for_values<V, KI, KO>(
  input: UsedAs<{ [KI]: V }>,
  transform: (value: V, key: () -> KI, active: Readable<boolean>) -> (KO, number?)
) -> () -> { KO }
```

Maps a table's values. The transform is handed the value, a *reader* for the key — an entry's key changes when the list reorders — and an `active` readable that goes false the moment the entry leaves the input.

> The transform may return a second value: a delay in seconds. With one, a removed entry stays in the output with `active` reading false, and is destroyed only once the delay elapses — which is how an entry animates itself out. Fluid's README puts this among the features "where Fluid has an advantage"; the comparison to Vide and Fusion is the library's claim, not this reference's finding.
>
> Two consequences of the implementation: every per-entry table is keyed by the **value**, so duplicate values collapse into one output, and the returned array is assembled by iterating that hash table, so its **order is not defined**. Reach for `for_keys` when values can repeat, and drive `LayoutOrder` from `key()` when order matters.

### `for_keys`

```luau
fluid.for_keys<K, VI, VO>(
  input: UsedAs<{ [K]: VI }>,
  transform: (value: () -> VI, key: K, active: Readable<boolean>) -> (VO, number?)
) -> () -> { VO }
```

Maps a table's **keys** — one output per key, which is what you want when values can repeat. The arguments are not `for_values`' in a different order: which one is a reader flips too. Here the *value* arrives as a reader and the *key* is passed plainly, the reverse of `for_values`.

> Everything else matches `for_values`: the same `active` readable, the same optional delay as a second return, and the same unordered output — this one keyed by the key, so duplicates are not a problem.

## Animation

### `spring`

```luau
fluid.spring<T>(input: UsedAs<T>, speed: number, damping: number)
  -> (() -> T, (SpringControls<T>) -> ())
```

Returns two things: a reader for the current value, and a control function for nudging the simulation directly.

> Returning a pair rather than an object is the shape to remember — `local position, control = fluid.spring(goal, 30, 1)`.
>
> Animatable types: `number`, `Vector3`, `Color3`, `UDim` and `UDim2`. Anything else raises `tried tweening <type>` at creation, so `Vector2`, `Rect` and `CFrame` are out — the near-inverse of `lerp`'s list. A spring cannot change data type once created.

### `lerp`

```luau
fluid.lerp<T>(goal: UsedAs<T>, takes: UsedAs<number>) -> Readable<T>
```

Moves towards a goal over a duration in seconds. Fluid's equivalent of a tween — but check the type first.

> Animatable types: `number`, `Vector3`, `Vector2`, `Color3`, `Rect` and `UDim`. **`UDim2` and `CFrame` are commented out upstream**, and there is no guard: a `UDim2` goal throws inside the animation step, and because the broken entry is never removed from the queue, *every later frame throws too*. `Position` and `Size` are `UDim2`, so for those use `spring`.

## Scheduling

### `provide_scheduler`

```luau
fluid.provide_scheduler() -> (delta_time: number) -> ()
```

Cancels Fluid's own `RunService.PreRender` loop and hands back its step function, so you can drive updates yourself. That step runs the scheduler, flushes deferred graph nodes, then advances `lerp` and `spring`, in that order.

> This is exactly how Kalo runs Fluid in your browser: no frame loop exists here, so `step(frames)` calls the function this returns.

### `graph`

```luau
fluid.graph.get_scope() -> Node?
```

The reactive graph module, exported for tooling. `get_scope()` returns the node the current thread is running inside, or `nil` at the top level; a scope whose `effect` field is not `false` is a reactive one.

> Not a stable API — it is the graph's internals, and it is the module the two scope-assertion errors come from. Useful for answering "am I inside a root?" while debugging; do not build on it.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Fluid](https://github.com/ffrostfall/fluid) | The library itself | 0.7.0-rc.7 | `fluid = "ffrostfall/fluid@=0.7.0-rc.7"` |
| [Wally](https://github.com/UpliftGames/wally) | Package manager; installs Fluid into `Packages/` | 0.3.2 | `rokit add UpliftGames/wally` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Syncs source files into Studio | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [Rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager; pins Rojo and Wally per project | 1.2.0 | `curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh \| bash` |
| [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) | Language server; reads Rojo's sourcemap output | 1.69.0 | `rokit add JohnnyMorganz/luau-lsp` |
| [Vide](https://github.com/centau/vide) | The library Fluid was rewritten from, and Kalo's other course on it | 0.4.1 | `vide = "centau/vide@0.4.1"` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **Anything calling 0.7 stable.** `wally.toml` says `0.7.0-rc.7`, and the 0.7 line carries no tag. The latest tagged stable release is **v0.6.5** (9 July 2026), which is a different library in places — names can still move before 1.0, and the changelog shows they have, since 0.7.0 removed dev groups outright. That changelog entry is one line, though: the RC line's git log also removed the action priority system outright and rewrote the graph and `bind` several times, so for this version the log is the real record.
- **Vide code pasted straight in.** Fluid kept Vide's shape, so `source`, `derive` and `effect` look identical — but the rewrites were, in the project's words, opinionated and breaking. `for_values` in particular takes a different transform, and Fluid applies **no per-class property defaults** where Vide does.
- **`fluid.spring(...)` used as one value.** It returns a reader *and* a control function. Assigning it to a single variable gets you only the reader, and the mistake is silent until you try to control the spring.
- **`fluid.bind(property, value)`.** The real order is `bind(instance, props)` — instance first, the same mixed table `create` takes second. The wrong order does not error; it writes `nil` into your source.
- **`_G.__DEV__` as a switch.** The README advertises it for development mode "at the cost of performance", but `dev_mode = true` is hardcoded in both `src/reactive/graph.luau` and `src/logging.luau`, and `__DEV__` is never read anywhere in the source. Every consumer pays the dev-mode cost — a coroutine per reactive evaluation — whether they set the flag or not.
- **`for_values`' output treated as ordered.** The array is assembled by iterating a hash table keyed by the input value, so it is neither in input order nor stable between runs — and two equal values collapse into one entry. Drive `LayoutOrder` from the transform's `key` reader, or use `for_keys` when values can repeat.
- **`show_delay` / `switch_delay` called like `show` / `switch`.** Passing plain values as branches is accepted and silently produces nonsense: every branch of a `_delay` variant must be a function `(active) -> (value, delay?)`, and the result is an **array** of the live branches rather than the branch itself.
