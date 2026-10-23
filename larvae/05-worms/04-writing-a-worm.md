# Writing one

*worm.toml declares what the worm claims, reports and offers, and two commands try it before it is installed.*

Larvae · Worms & Extensions · 11 min

---

A worm declares itself in a `worm.toml`. Here is the head of the real one from `luaux-worm`, which is the file larvae reads to know what it is dealing with.

**worm.toml, the head**

```toml
name  = "luaux"
api   = 1
form  = "native"
entry = "luaux-worm"

[frontend]
claims        = [".luaux"]
fmt           = true
inherit_lints = true
shared        = true
```

`claims` is the list of extensions larvae will hand to this worm. `api` is the protocol version the worm speaks.

`shared = true` is worth understanding. It means other worms may also serve `.luaux` files: their Luau-reading lints run over this worm's byte-true shadow of the file, so a project-wide convention still holds inside markup.

## Declaring lints

**Each lint, with a default**

```toml
[lints.compile_error]
description = "A problem that stops the luaux compiler, such as an element that no tag closes"
default     = "deny"

[lints.static_conditional_child]
description = "A conditional child that the code builds one time, so the UI does not update"
default     = "warn"
```

> **There is no `error` level — `deny` is the one that fails the run**
>
> `luaux-worm`'s manifest says "a level is allow, warn, or deny; there is no error". That is about the *spelling*: selene has a level called `error` and larvae does not, so `deny` is what a worm writes for the level that stops a build. It is not a claim that a worm gets three of larvae's four levels.
>
> A worm's `default` is an ordinary larvae `Level`, so `info` is available to it like any other, and it defaults to `warn` when the manifest says nothing — the same as most builtins.

Formatter options and settings are declared the same way — `[fmt.<name>]` with a type, permitted values and a default, and `[options.<name>]` for whatever the worm reads from its `config` table. `luaux-worm` has exactly one of the latter, pointing at the `luaux.toml` that holds everything the compiler itself cares about.

**A worm that lints plain Luau**

```toml
name       = "privacy"
api        = 1
form       = "luau"
entry      = "init.luau"
lints_luau = true      # run this worm's lint over every plain Luau file

[lints.private_access]
description = "A field whose name starts with _ read from outside its class"
default     = "warn"
```

A worm need not claim anything. `lints_luau = true` runs its lint over ordinary `.luau` files, after the builtin lints and under the same levels and allow comments — and over a claimed file's shadow too, when its worm sets `shared`. With no `[frontend]`, no rules and no `lints_luau`, larvae refuses the manifest: it would never run.

> **Recognising older code**
>
> A manifest written before 0.10 may carry an `[lsp]` table — `resolve`, `declarations`, `respond`, `serves_luau` — that fed larvae's type analyzer. The analyzer is gone, so the table still loads, larvae prints one note per worm, and the hooks do nothing. Delete it. A worm's editor answers now are its findings, its layout and its code actions.

Iterating on a worm needs no install at all: **`larvae worm run <dir> <file>`** passes one file through a worm directory, with `--fmt` or `--lint` to exercise those paths instead, and **`larvae worm info <dir>`** reports what it declares. When you are ready to wire it into a project, name it by `path` rather than `repo`. The rest of the family is `add`, `install`, `claims`, `remove` and `types` — the last writes the Luau type definitions a worm author codes against.

> **Next: the language on the other side**
>
> `luaux-worm` exists to serve LuauX, JSX-style markup that compiles to Luau. Kalo's **LuauX** course covers the language itself — and now that you know how worms work, its tooling story is one sentence rather than a chapter.

**What you can do now**

- Explain why a worm returns structure rather than formatted text, and findings without levels.
- Weigh the three forms, and know that `native` buys speed by giving up the sandbox.
- Install a worm and configure everything it adds under its own key — remembering the dotted `<worm>.<lint>` name everywhere else.
- Read a `worm.toml` and know what the worm claims, reports and offers — including a worm that claims nothing and lints plain Luau.
- Iterate on one locally with `larvae worm run` and `larvae worm info`, no install required.
