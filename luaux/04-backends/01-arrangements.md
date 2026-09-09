# Two arrangements

*Four libraries, two shapes of constructor call, and why a runtime adapter was rejected.*

LuauX · Backends · 11 min

---

LuauX has to reach four Roblox UI libraries. It knows about none of them. What it knows is that they differ along four axes — and, crucially, that those differences are not all of the same kind.

|  | arrangement | children | events | reactive value |
| --- | --- | --- | --- | --- |
| Vide | `F(class)(props)` | numeric keys in props | string key | function |
| Fluid | `F(class)(props)` | numeric keys in props | string key | function |
| Fusion | `F(class)(props)` | `[Children]` in props | `[OnEvent(name)]` | `StateObject` |
| React | `F(class, props, children)` | third argument | `[Event.Name]` | none |

The table from `docs/adr/0003-backends-own-arrangement.md`. If you have used one of these, read your own row.

**Two shapes of call**

```luau
-- table: F(class)(props) — Vide, Fluid, Fusion
create("Frame")({ Size = size, child })

-- element: F(class, props, children) — React
React.createElement("Frame", { Size = size }, { child })
```

Three of the four take one table. React takes three arguments. **That** difference changes the shape of the emitted call; the others only change what goes where inside a table LuauX was already building.

> **The rule, in one sentence**
>
> A **`[factory]` variable** changes what goes where inside one props table passed to a curried constructor. Anything that changes the **arity or arrangement** of the constructor call needs a **backend**.

**Same arrangement, different contents**

```luau
-- Vide: children are numeric keys
create("Frame")({ Size = size, child })

-- Fusion: the same call, children under a named key
scope:New("Frame")({ Size = size, [Children] = { child } })
```

Fusion moves the children and changes how an event is spelled. Neither touches the arity of the call, so neither needs a backend.

So there are exactly two backends, and they are named for the shape rather than for a library — `table` for `F(class)(props)` and `element` for `F(class, props, children)`. Naming one `React` would bake a library into the compiler and make every library without a preset second-class.

## Why not a runtime adapter?

The obvious alternative was to emit one shape everywhere and ship a small per-library adapter that translates at runtime. It was rejected:

- **It reintroduces a runtime.** LuauX refuses to resolve module paths, because a require is location-dependent and no configured string is right for every file. An adapter would be a module every generated file must require.
- **The value type differs, and no adapter can hide it.** Under Vide, Fusion and Fluid `<Frame/>` *is* an Instance. Under React it is an inert description. An adapter makes the call portable and leaves your surrounding code exactly as unportable as before.
- **Reactivity cannot be adapted.** An adapter handed `Size = function() … end` has no scope to build a Fusion `Computed` with, and no hook site to register React state at.

**Key points**

- Two backends, named for shape: `table` is `F(class)(props)`, `element` is `F(class, props, children)`.
- Anything that changes the arity or arrangement of the call is a backend; anything inside one table is a `[factory]` key.
- Fusion is the table backend plus settings; React is the element backend.
- A runtime adapter was rejected: it reintroduces a runtime and cannot hide that an element is an Instance under three targets and a description under the fourth.
