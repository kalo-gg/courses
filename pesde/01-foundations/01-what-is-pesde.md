# What is pesde?

*Why a Luau package manager declares a target, and where its packages come from.*

pesde · pesde Foundations · 9 min

---

Luau runs in more than one place. It runs inside **Roblox**, standalone through the **luau** CLI, and inside **Lune** for scripts and tooling. A library that works in one of those may be meaningless in another — `game:GetService` outside Roblox, `@lune/fs` inside it.

pesde takes that seriously. Every package declares the environment it is for, and a single package name at a single version may exist several times, once per environment. Installing resolves the version **and** the target.

**pesde.toml**

```toml
name = "you/hello_pesde"
version = "0.1.0"
license = "MIT"

[target]
environment = "luau"
```

One manifest per package. `environment` is the target; everything else in this course hangs off it.

## Four targets, three runtimes

| `environment` | What it means | Installs into |
| --- | --- | --- |
| `luau` | Standalone Luau, run with the `luau` CLI | `luau_packages/` |
| `lune` | Luau needing the Lune runtime | `lune_packages/` |
| `roblox` | Luau that must run in Roblox | `roblox_packages/` |
| `roblox_server` | As `roblox`, but server-side only | `roblox_server_packages/` |

Four targets, three runtimes: `roblox` and `roblox_server` are the same runtime linked differently, and each still gets its own directory.

**Where install puts them**

```text
luau_packages/           # environment = "luau"
lune_packages/           # environment = "lune"
roblox_packages/         # environment = "roblox"
roblox_server_packages/  # environment = "roblox_server"
```

One directory per target, beside your code. The name is the target's, not the source's — a Wally package installed into a Roblox project still lands in `roblox_packages/`.

> **Where the packages actually come from**
>
> The default registry is `pesde-pkg/index` on GitHub — a git repository of metadata, with the tarballs served by the registry API. You can add more indices, or run your own, and a project can pull from several at once.

pesde also installs **Wally** packages, and **git** repositories, and directories on your disk. Those are covered in The Manifest; the important thing now is that they all end up in the same dependency graph and the same lockfile.

**Key points**

- Every package declares an `environment`; the same name and version may be published once per environment.
- Installing resolves a version **and** a target, and links into a directory named after the target.
- The default registry is the `pesde-pkg/index` repository; Wally, git and local directories feed the same graph and lockfile.
- A dependency with no `target` takes yours.
