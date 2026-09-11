# Naming a package

*The spec is scope/name@requirement, the alias is yours to choose, and the version half is required.*

Wally · The Manifest · 10 min

---

Everything you will ever write in a dependency table has the same shape: a name you choose on the left, and a **package spec** on the right. The spec is where Wally does its work, and it has exactly three parts — `scope/name@version-requirement` — with no optional ones.

**One dependency line**

```toml
[dependencies]
Signal = "sleitnick/signal@^2.0.0"
```

`Signal` is yours. `sleitnick/signal` identifies the package in the index. `^2.0.0` says which versions of it you will accept.

| Part | In the example | Who decides it |
| --- | --- | --- |
| The alias | `Signal` | You — it is free-form and means nothing to the registry |
| The scope | `sleitnick` | The publisher, claimed on first publish |
| The name | `signal` | The publisher, in the package's own `wally.toml` |
| The requirement | `^2.0.0` | You — a range of versions, not one version |

Only the middle two are fixed. The alias and the requirement are the parts you are choosing when you write the line.

The alias has nothing to do with the package's own name. It becomes the name of the **link module** Wally writes on disk, which is what you `require`, so choosing `Signal` means the rest of your codebase says `Packages.Signal` — and choosing `Sig` would mean `Packages.Sig` instead.

**What the alias becomes**

```text
Packages/
  Signal.lua                       <- the link module, named after your alias
  _Index/
    sleitnick_signal@2.0.2/
      signal/                      <- the package, named by its publisher
```

One file per alias at the top, the real contents flat underneath. Module 3 takes this tree apart properly.

## The version half is not optional

This is the part people trip over first. `Signal = "sleitnick/signal"` is not a shorthand for "any version" — it is a **parse error**, and the manifest is rejected before anything is resolved. That is deliberate. A missing requirement in a package manager that guessed would silently become a wildcard, and a wildcard is almost never what someone meant to write.

**Four ways to get it wrong**

```toml
[dependencies]
Signal = "sleitnick/signal"          # no requirement — parse error
Promise = "promise@^4.0.0"           # no scope
Roact = "Roblox/Roact@^1.4.0"        # uppercase is not a legal name
Trove = "sleitnick/trove^1.0.0"      # the @ is missing
```

All four fail before anything is fetched, which is the good outcome: none of them can turn into a wrong install.

The name rules from Module 1 apply to the right-hand side too: lowercase ASCII letters, digits and hyphens, 1–64 characters per half. The **alias** on the left is not bound by them, and the convention is to write it the way you want the instance to read — `Signal`, `ProfileStore`, `TestEZ` — because that is exactly what it will read as.

> **If you want a specific version, say so**
>
> Being forced to write a requirement does not mean being forced to write a loose one. `"biff/minimal@=1.2.0"` is a perfectly good spec and pins exactly one version. The next lesson is about choosing between the two.

**Key points**

- A package spec is `scope/name@version-requirement`, and all three parts are required.
- The alias on the left is free-form and becomes the name of the link module you `require`.
- Renaming an alias renames the instance, so it is a codebase-wide change, not a manifest tidy-up.
- Omitting the requirement is a parse error on purpose: a missing requirement must never become a silent wildcard.
- Scope and name follow Module 1's rules — lowercase letters, digits and hyphens.
