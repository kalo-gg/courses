# Scopes and the reactive graph

*Stable scopes hold effects; destroying a root destroys everything created inside it.*

Vide · Sources, Effects & Scopes · 11 min

---

A connection to a signal has to be disconnected eventually, or it keeps whatever it references alive. Effects have the same problem, and disconnecting each one by hand would be tedious. Vide's answer is **scopes**: every effect belongs to the scope it was created in, and destroying a scope destroys everything inside it.

## Two kinds of scope

| Scope | Created by | Reruns? | May contain |
| --- | --- | --- | --- |
| **Stable** | `root`, `show`, `switch`, `indexes`, `values`, `mount` (and `untrack`, with an asterisk below) | Never | Stable and reactive scopes |
| **Reactive** | `effect`, `derive` | When a tracked source changes | `untrack`, `root` or `mount` only (the dynamic helpers create effects, so call them from a stable scope) |

The rules are short: a scope must be created inside another scope; a reactive scope cannot be created directly inside another reactive scope; and `root` is the exception that starts a tree, handing you a destructor to end it. When a reactive scope reruns, everything created inside it during the previous run is destroyed first.

> **`untrack` does not really create a scope**
>
> It is listed with the stable scopes because it lets you create an effect from inside an effect, but that is not how it works. `untrack` creates no node at all: it flips the **current** scope's `effect` flag off, runs your function, and flips it back. Nothing is pushed, nothing is owned.
>
> The difference is visible: anything you create inside `untrack` belongs to the surrounding effect, so it is destroyed and rebuilt on that effect's next rerun. A nested `root` is a genuine new scope that survives until you destroy it yourself. When you need something to outlive the effect that created it, reach for `root`, not `untrack`. (Vide's own docs describe `untrack` as "runs a function in a new stable scope", which is the source of the confusion.)

```luau
local root, source, effect = vide.root, vide.source, vide.effect

local count = source(0)

local function setup()
	effect(function()
		print(count())
	end)
end

-- setup() here would error: no stable scope to hold the effect

local destroy = root(setup) -- prints 0

count(1) -- prints 1
count(2) -- prints 2

destroy()

count(3) -- the effect is gone; nothing prints
```

> **Why the rule about reactive-in-reactive?**
>
> If an effect created another effect directly, every rerun of the outer one would destroy and recreate the inner one, which is almost never what you want and easy to do by accident. Requiring you to say which escape hatch you mean makes the choice explicit.
>
> The two are not interchangeable. `untrack` only suspends tracking, so the inner effect is still owned by the outer one and *is* destroyed and recreated on every rerun — you have asked for that behavior deliberately instead of by accident. A nested `root` gives the inner effect its own lifetime, independent of the outer effect's reruns, and hands you a destructor to end it.

## The reactive graph

It helps to picture sources and scopes as a graph. Sources are nodes with no parent (they can live outside any scope). Effects and derived sources are nodes inside a scope, with edges from the sources they read. Destroying a root deletes the subtree under it, leaving the sources untouched.

**Graph for the example above**

```text
count ──▶ [root: effect(print)]

after destroy():

count      (no dependents; writes do nothing)
```

**Key points**

- Every effect lives in a scope; destroying the scope destroys the effect.
- `root` creates a stable scope and returns its destructor first; from here on, snippets assume they run inside a `root` (or `mount`).
- Reactive scopes (`effect`, `derive`) cannot nest directly; put `untrack` (or a nested `root`) between them. The dynamic helpers create effects too, so call them from a stable scope.
- `untrack` creates no scope — it suspends tracking in the current one, so what you make inside it is still owned by the surrounding effect. Use a nested `root` when you want an independent lifetime.
- Sources live outside scopes and survive destruction; only their dependents go.
