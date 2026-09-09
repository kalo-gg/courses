# Vide reference

*Vide 0.4.1, released July 11, 2026. Checked on September 7, 2026.*

---

## Reactivity: core

Sources hold data, effects react to it, and scopes decide how long that reaction lives.

### `source`

```luau
vide.source<T>(value: T) -> Source<T>
source() -> T   -- read
source(v) -> T  -- write
```

Creates a source: a function that returns its value when called with no arguments and sets it when called with one. Reading inside a reactive scope tracks the source; writing reruns dependents unless the value is unchanged.

> Tables count as changed on every write unless frozen, so `src(t)` after `table.insert(t, x)` does rerun effects.

### `effect`

```luau
vide.effect<T>(fn: (T) -> T, initial?: T)
```

Runs `fn` once immediately in a new reactive scope, then reruns it whenever a source it read updates. Must be called inside a stable scope (root, show, indexes …). With the two-argument form `fn` receives the value it returned last time.

### `derive`

```luau
vide.derive<T>(fn: () -> T) -> () -> T
```

Creates a cached, read-only source computed from other sources. Recomputes only when a dependency updates, and does not rerun dependents if the new value equals the old one.

> Prefer `derive` over an effect that writes to a source: it cannot create update loops.

### `root`

```luau
vide.root<T...>(fn: (destroy: () -> ()) -> T...) -> (() -> (), T...)
```

Runs `fn` in a new stable scope. Returns the destructor first, then whatever `fn` returned. Destroying the root destroys every scope created inside it and runs their cleanups.

> If `fn` errors, the scope is destroyed automatically and the error is rethrown.

### `mount`

```luau
vide.mount<T>(component: () -> T, target: Instance?) -> () -> ()
```

Runs a component in a new stable scope and, if `target` is given, parents the returned instance to it. Returns the destructor.

## Reactivity: utility

### `cleanup`

```luau
vide.cleanup(fn | connection | instance | thread | { Destroy | Disconnect })
```

Queues something to run when the current scope reruns or is destroyed: a function is called, a connection disconnected, an instance destroyed, a thread canceled. Any table with a `destroy`/`Destroy` or `disconnect`/`Disconnect` method works too.

> A thread is canceled with `task.cancel`, not closed — that errors on a thread that is neither suspended nor scheduled, so wrap in a function if the thread may already have finished. In the Kalo sandbox the bundle is built by `scripts/build-vide.mjs`, which rewrites this branch to `coroutine.close(obj)`. `coroutine.close` returns quietly on a thread that has already finished, where `task.cancel` errors, so a snippet that cleans up a completed thread passes in the browser and fails in Studio.

### `untrack`

```luau
vide.untrack<T>(fn: () -> T) -> T
```

Suspends tracking in the current scope while `fn` runs and returns its result, so sources read inside do not subscribe the surrounding effect. Anything created inside still belongs to that effect and goes with its reruns.

### `read`

```luau
vide.read<T>(value: T | () -> T) -> T
```

Returns `value()` if it is a function, otherwise `value`. Lets a component accept either a plain value or a source for the same prop.

### `batch`

```luau
vide.batch(fn: () -> ())
```

Runs `fn` and defers every effect triggered by source writes inside it until `fn` returns, so several writes cause one rerun.

### `context`

```luau
vide.context<T>(default: T) -> Context<T>
context() -> T                 -- read
context(value, fn) -> R        -- provide
```

Creates a context. Calling it with no arguments returns the nearest provided value (or the default); calling it with a value and a function runs the function in a new stable scope where that value is provided, and returns the function's result.

## Dynamic scopes

Create and destroy UI in response to sources. Each returns a source you can parent as a child.

### `show`

```luau
vide.show(source: () -> unknown, component: (source, present) -> T, fallback?: (present) -> U) -> () -> T | U | { T | U } | nil
```

Runs `component` in a stable scope while `source()` is truthy and destroys it when it becomes falsey; shows `fallback` meanwhile. Return a second value (seconds) from the component to delay destruction, for exit animations; while both exist the source holds them in an array.

### `switch`

```luau
vide.switch(source: () -> K)(map: { [K]: (present) -> V }) -> () -> V | { V } | nil
```

Picks a component constructor from `map` by the source's value, runs it in a stable scope, and swaps (destroying the old scope) when the value changes. Map functions receive a `present` source and may return a delay; while an outgoing scope is still waiting out its delay the source holds both objects in an array, exactly like `show`.

### `indexes`

```luau
vide.indexes(source: () -> { [K]: V }, fn: (value: () -> V, index: K, present) -> T) -> () -> { T }
```

Creates one stable scope per table index. New indexes run `fn`, removed indexes destroy their scope, and a changed value at an existing index just updates that scope's `value` source. `fn` also receives a `present` source and may return `(instance, delaySeconds)`.

### `values`

```luau
vide.values(source: () -> { [K]: V }, fn: (value: V, index: () -> K, present) -> T) -> () -> { T }
```

Like `indexes` but keyed by value: each distinct value gets a scope, and a value that moves to another index just updates its `index` source. Duplicate values are an error in strict mode.

### `present`

```luau
present: Source<boolean>
-- 2nd argument of a show/switch component, 3rd of an indexes/values component
```

The source every dynamic-scope component receives: `true` while its scope is live, `false` from the moment the scope is scheduled for destruction, and back to `true` if it is revived during its delay.

> Bind exit animations to it and return `instance, seconds` from the component to get the delay (Delayed destruction).

## Element creation

### `create`

```luau
vide.create("ClassName")(props) -> Instance
vide.create(instance)(props) -> Instance  -- clones
```

Returns a constructor for the class (or for clones of the instance). Calling it with a property table sets string keys as properties, connects functions on event names, binds other functions as implicit effects, and parents numeric-key instances as children.

> The old `create(class, props)` two-argument form is deprecated in 0.4.1; use `create "Class" { … }`. `create(instance)` clones at runtime, but 0.4.1's exported type only accepts a class-name string, so the clone form does not typecheck — cast it, or use `apply` on a clone you made yourself. A function in an unkeyed slot is a child binding: it may return one instance, an array (nested arrays are fine) or nil; Vide parents what it returns and unparents what disappears between runs.

### `apply`

```luau
vide.apply(instance)(props) -> Instance
```

Applies a property table to an existing instance with the same rules as `create`. Marked temporary in the source.

### `action`

```luau
vide.action(fn: (Instance) -> (), priority: number?) -> Action
```

Wraps a callback that runs with the instance after its properties are set and events connected. Lower priority values run first for small positive numbers (default 1); the map is not sorted.

### `changed`

```luau
vide.changed(property: string, fn: (value) -> ()) -> Action
```

An action that calls `fn` with the property's value immediately and again whenever the property changes. The connection is cleaned up with the scope.

## Animation

### `spring`

```luau
vide.spring<T>(source: () -> T, period: number?, damping: number?) -> (() -> T, SpringControl<T>)
```

Returns a source that moves towards the input value with spring physics (period in seconds for one undamped cycle, damping ratio 1 = critically damped), plus a control function for setting position, velocity or an impulse.

> Animates number, CFrame, Color3, UDim, UDim2, Vector2, Vector3 and Rect — and, undocumented upstream, arrays of up to four numbers (`{ number }`, `spring.luau:79`). Springs advance from RunService.Heartbeat; in the Kalo sandbox call `step()`.

### `step`

```luau
vide.step(dt: number)
```

Advances the spring solver and the delayed-destruction timers by `dt` seconds. This is a real public API, not a sandbox helper: Kalo's `step()` calls it for you.

> The first call permanently disconnects Vide's own Heartbeat connection, so once you step manually you own the frame loop for the rest of the session. Useful for tests and for running Vide outside Roblox.

### `control`

```luau
local value, control = spring(…)
control({ position = v })
control({ velocity = v })
control({ impulse = v })
```

The second value `spring` returns: moves the solver without resetting it — set the position, set the velocity, or add an impulse (`spring.luau`'s `SpringSettings<T>`).

> Writing the spring source itself (`value(v)`) snaps the output there and clears the velocity; the control keeps the motion.

## Flags

### `vide.strict`

```luau
vide.strict: boolean
```

Strict mode: reactive scopes run twice per update to catch impure computations, yields inside scopes error, and error traces improve. On by default at O0/O1 (Studio), off at O2 (production).

### `vide.defaults`

```luau
vide.defaults: boolean
```

When true (default), `create` applies Vide's sensible defaults to new instances, such as `BorderSizePixel = 0` on frames and `AutoButtonColor = false` on buttons.

### `vide.defer_nested_properties`

```luau
vide.defer_nested_properties: boolean
```

When true (default), a nested property table inside a `create` call is pushed onto a stack and applied in a later pass rather than recursed into immediately, so the outer table's own properties are all set first.

### `vide.version`

```luau
vide.version: { major: number, minor: number, patch: number }
```

The loaded library version.

## Exported types

Vide exports two types for annotating props and modules; both have lowercase aliases.

### `vide.Source<T>`

```luau
export type Source<T> = (() -> T) & ((value: T) -> T)
```

The type of a source: callable with no arguments to read, with one to write. Also exported as `vide.source<T>`.

> Annotate props that must be live as `() -> T` (read-only) or `vide.Source<T>` (writable).

### `vide.Context<T>`

```luau
export type Context<T> = (() -> T) & (<U>(T, () -> U) -> U)
```

The type `context()` returns: read with no arguments, provide with a value and a function. Also exported as `vide.context<T>`.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Vide](https://github.com/centau/vide) | Reactive UI library | 0.4.1 | `centau/vide (wally) · build.rbxm on GitHub releases` |
| [pesde](https://github.com/pesde-pkg/pesde) | Package manager (Vide's own toolchain lists it first) | 0.6.x | `rokit add pesde-pkg/pesde` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Sync source files into Studio | 7.x | `rokit add rojo-rbx/rojo` |
| [Wally](https://github.com/UpliftGames/wally) | Package manager | 0.3.x | `rokit add UpliftGames/wally` |
| [Rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager the install strings above use | 1.2.0 | `curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh \| bash` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- `create("Frame", { … })` with two arguments is deprecated since 0.4.1; write `create "Frame" { … }` (the constructor returned by `create` takes the property table).
- Since 0.4.0 `spring()` returns two values: the animated source and a control function for position, velocity and impulse.
- Since 0.4.0 `show()` passes the current condition value to the component as its first argument (a source) and `present` as the second, and dynamic scopes may return a delay in seconds to postpone destruction.
- `root()` has returned the destructor as its first value since 0.3.0; older tutorials show it returned only the callback's result.
- `context()` exists since 0.3.0; before that, values were threaded through props.
- `cleanup()` accepts connections and instances directly since 0.1.1 and threads since 0.4.0; you do not need to wrap them in functions.
- `branch()` is still commented out of the public API in 0.4.1 even though the source exists.
- 0.4.1's typed class list (`Instances` in `create.luau`) covers only 19 classes. `create "UIPadding" { … }` runs fine but does not typecheck; `UIPadding`, `UIStroke`, `UIScale`, `ViewportFrame`, `ProximityPrompt` and friends were added after the tag and are unreleased.
- The strict-mode docs list a check for "`indexes()` and `values()` outputting primitive values". No such check exists in 0.4.1 — `indexes.luau` and `values.luau` only check for duplicate values, so a primitive-returning component fails later and less clearly.
- Vide is pre-1.0 and every GitHub release is flagged as a prerelease, so `/releases/latest` returns nothing. Each minor has carried breaking changes — read the changelog before bumping.
- Releases before 0.4.0 have no downloadable assets at all, and the file attached to 0.4.0 and 0.4.1 is named `build.rbxm`, not `vide.rbxm`.
- Vide's docs describe `untrack` as running its function in a new stable scope. In 0.4.1 it creates no scope: it turns the current scope's `effect` flag off while your function runs, so anything created inside still belongs to the surrounding effect and is destroyed on its next rerun (`untrack.luau`).
