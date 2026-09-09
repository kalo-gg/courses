# The sync config generator

*Why Roblox needs a second step, the scripts package that provides it, and the install that makes it real.*

pesde · Roblox Projects · 11 min

---

For every other target, installing is the whole job: the files are on disk and `require` finds them. Roblox has a second step — the files must be described to a sync tool before they exist in the datamodel at all.

pesde does not do that itself. It runs a script you provide, called the **`roblox_sync_config_generator`**.

> **Scripts packages are on their way out**
>
> pesde's own guide opens with a caution: scripts packages exist today, but their job is expected to move to binary packages before 1.0. Learn the mechanism — it is how 0.7.3 works and how every Roblox project is set up right now — but do not build tooling that assumes it will still be here at 1.0.

**pesde init, for Roblox**

```bash
pesde init

# what environment are you targeting for your package? roblox
# which scripts package do you want to use? pesde/scripts_rojo

# initialized project
# tip: run `install` to fully finish setup
```

Choosing a Roblox target skips the *would you like to setup Roblox compatibility scripts?* question that other targets get — you are taken straight to the picker. Decline it (pick the custom option and leave the name empty) and init warns `no scripts package configured, this can cause issues with Roblox compatibility` and asks whether to initialize regardless.

**What that writes**

```toml
[target]
environment = "roblox"
lib = "src/init.luau"
build_files = ["src"]

[scripts]
roblox_sync_config_generator = ".pesde/scripts/roblox_sync_config_generator.luau"
sourcemap_generator = ".pesde/scripts/sourcemap_generator.luau"

[dev_dependencies]
scripts = { name = "pesde/scripts_rojo", version = "^0.2.0", target = "lune" }
rojo = { name = "pesde/rojo", version = "^7.5.1", target = "lune" }

[engines]
pesde = "^0.7.0"
lune = "^0.10.2"
```

`[scripts]` entries are run with **Lune**, not by Roblox. The path is `.pesde/<alias>/<name>.luau`, where the alias is the scripts package's — `scripts` here, because that is the alias init used in `[dev_dependencies]`.

## init copies nothing

Read those four sections together and the mechanism is obvious. `init` **copies nothing**: it adds the scripts package as a dev dependency under the alias `scripts`, writes `[scripts]` paths that point at where that alias will be linked, copies the scripts package's peer dependencies (Rojo itself) and its `[engines]` requirements across, and stops. Nothing exists at `.pesde/scripts/` yet.

**The step init tells you to take**

```bash
pesde install
```

This is what creates `.pesde/scripts/` and puts the two generator scripts in it. Until you run it both `[scripts]` paths point at nothing — and pesde will not fail an install over that. The configuration simply never gets written, which is a far worse failure to debug than an error would be.

> **`pesde install` before `rojo serve`**
>
> The sync configuration for your dependencies is generated *during* install, into the installed packages. So the order is load-bearing: install, then serve. A fresh clone that runs `rojo serve` first sees a `roblox_packages` tree with no configuration in it.

> **Not using Rojo?**
>
> Look for your tool's scripts package on the registry — the [`pesde-pkg/scripts`](https://github.com/pesde-pkg/scripts) repository collects them. If yours is missing, you can write the generator yourself and put its path in the manifest, then send a pull request so the next person does not have to.

**Key points**

- pesde installs files; the `roblox_sync_config_generator` you name in `[scripts]` writes the sync tool's config for each installed package.
- `pesde init` adds the scripts package as the dev dependency `scripts` and points `[scripts]` at `.pesde/scripts/…`; nothing exists there until `pesde install`.
- A wrong script path does not error — the config is simply never written.
- Install, then serve.
