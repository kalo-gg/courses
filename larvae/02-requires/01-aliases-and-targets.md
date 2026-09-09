# Aliases and targets

*One source spelling, three possible outputs, and the indexing style that decides whether a chain waits.*

Larvae · Rewriting Requires · 10 min

---

An alias gives a location a short name. You write the short name everywhere, and larvae expands it when it builds — so moving a folder is a one-line change to the config rather than a search across the project.

**Naming places**

```toml
[aliases]
pkg = "@game/ReplicatedStorage/Packages"
shared = "./src/shared"
```

A value is either a DataModel path (`@game/…`) or a filesystem path (`./…`). Names match case-insensitively, and `@self` and `@game` are reserved.

> **You may already have these**
>
> `[aliases]` is merged over `.luaurc` key by key. A project with aliases already declared there needs none of this section — larvae reads them.

Names match case-insensitively, and two are reserved: `@game` is the root every DataModel path starts from, and `@self` is larvae's own. An alias value is one of two things — a DataModel path beginning `@game/`, or a filesystem path beginning `./` — and which one you can use depends on where the build is going, which is what the target decides.

## The three targets

| `target` | Emits | For |
| --- | --- | --- |
| `roblox-string` | Roblox's string require form | Most Roblox projects |
| `path` | Filesystem paths | Lune, and anything running outside Roblox |
| `roblox-instance` | An instance chain | Places that need the instance form, and older codebases |

With `roblox-instance` there is a second decision: how the emitted chain indexes each child. `indexing_style` picks between `find_first_child`, `wait_for_child` and `property` — which matters because a client requiring something that has not replicated yet is a real failure mode, and `WaitForChild` is the shape that survives it.

**The default does not wait**

```toml
[requires]
target = "roblox-instance"
# indexing_style = "find_first_child"   # the default: script.Parent:FindFirstChild("x")
indexing_style = "wait_for_child"       # script.Parent:WaitForChild("x")
# indexing_style = "property"           # script.Parent.x
```

`find_first_child` is the default, not `wait_for_child` — `larvae.example.toml` lists them in the other order, which reads like the opposite. This is also why `[check] early_require` warns out of the box: on an untouched `roblox-instance` project it is warning about larvae's own default. The require gate lesson picks that up.

**Key points**

- `[aliases]` names a place once; it is merged over `.luaurc` key by key, so a project with aliases there needs none of it.
- A value is a DataModel path (`@game/…`) or a filesystem path (`./…`); `@game` and `@self` are reserved.
- `roblox-string`, `path`, `roblox-instance`: which runtime the build is for is a property of the build, not the code.
- `indexing_style` defaults to `find_first_child`, which does not wait; `wait_for_child` is the shape that survives an early client require.
