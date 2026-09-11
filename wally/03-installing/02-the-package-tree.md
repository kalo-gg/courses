# What lands on disk

*Three directories of one-line link modules sitting over a flat index of every resolved package.*

Wally · Installing and the Tree · 10 min

---

Install writes up to three directories — `Packages/`, `ServerPackages/` and `DevPackages/`, one per realm — and each has the same two-layer shape. At the top are **link modules**, one per alias you wrote. Underneath is `_Index`, a flat directory holding the real source of every resolved package, direct and transitive alike.

**Packages/ and ServerPackages/ after an install**

```text
Packages/
  Minimal.lua
  Signal.lua
  _Index/
    biff_minimal@0.1.0/
      minimal/
        init.luau
        default.project.json
    sleitnick_signal@2.0.0/
      signal/
        init.luau
ServerPackages/
  ProfileStore.lua
  _Index/
    biff_profile-store@1.0.0/
      profile-store/
        init.luau
```

Nothing is nested by dependency. Every resolved package sits at the same depth in `_Index`, however deep in the graph it was found.

An `_Index` entry is `<scope>_<name>@<version>/<name>/`. Two details to hold on to: the scope and name are joined with an **underscore**, not the slash you wrote in the manifest, and the directory inside is the package's **short name** — `minimal`, not `biff/minimal` and not your alias.

**Packages/Minimal.lua**

```luau
return require(script.Parent._Index["biff_minimal@0.1.0"]["minimal"])
```

The entire file. A link module is one line: find the real package in `_Index` and hand it back.

## Three kinds of link

Every link is that same line with a different way of reaching `_Index`. From the **root** of a package directory, `script.Parent._Index` is next door. From **inside** the index — a package linking to a sibling it depends on — the same expression still works, because the sibling is in the same flat directory. **Across realms** it cannot: a server package reaching a shared one is in a different directory tree entirely, and no number of `script.Parent`s will get there.

**ServerPackages/_Index/biff_profile-store@1.0.0/Signal.lua**

```luau
return require(game.ReplicatedStorage.Packages._Index["sleitnick_signal@2.0.0"]["signal"])
```

So Wally pastes your `[place]` value in as raw Luau. `shared-packages = "game.ReplicatedStorage.Packages"` becomes the first half of that expression, which is why a missing `[place]` means the link cannot be written at all.

> **Link files are `.lua`, not `.luau`**
>
> Every link module Wally writes ends in `.lua`. A pull request to change that has been open for three years. It matters more than it sounds: editors and analysers key on the extension, so these files fall outside whatever your `.luau` tooling is configured for, and a project that assumes `--!strict` everywhere finds a layer of untyped indirection between it and every package it depends on.

## What a consumer writes

**Key points**

- Each realm directory has two layers: link modules at the top, every resolved package flat underneath in `_Index`.
- An index entry is `<scope>_<name>@<version>/<name>/` — an underscore where the manifest had a slash, and the short name inside.
- A link module is one line, `return require(...)`, and its file name is the alias you chose.
- A cross-realm link is written by interpolating your `[place]` strings as raw Luau, so without `[place]` it cannot exist.
- Wally writes link files as `.lua`, not `.luau`, which trips up editors and strict-mode expectations.
