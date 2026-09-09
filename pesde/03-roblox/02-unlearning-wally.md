# Unlearning Wally

*The project file a pesde package must not have, what the consumer's generator writes instead, and the tree that results.*

pesde · Roblox Projects · 10 min

---

If you have published a Wally package, you have written this file, and it is the standard thing to do there:

**default.project.json — the Wally convention**

```json
{
	"tree": {
		"$path": "src"
	}
}
```

It collapses `src` into the package itself, so the package *is* the contents of `src`.

> **A pesde package must not have a `default.project.json`**
>
> pesde **forbids** one in a published package, and ignores them in git dependencies. Instead you declare `build_files`, a 1:1 match between the file system and the structure inside Roblox. The reason is not tidiness: a project file bakes Rojo into your package, and `build_files` lets the *consumer* choose their own sync tool. It also stops a package being published with a broken shape — one configured as a DataModel, say.

**What replaces it**

```toml
[target]
environment = "roblox"
lib = "src/init.luau"
build_files = ["src"]
```

Top-level files and directories to sync. You never list `roblox_packages` — that is always synced.

Your consumer's `roblox_sync_config_generator` reads `build_files` and writes the configuration their tool needs. A Rojo user ends up with a generated `default.project.json` mapping `src` and `roblox_packages` — the file you were forbidden from writing, produced for them, in their tool's dialect:

**What `pesde/scripts_rojo` writes into each installed package**

```json
{
  "name": "1.0.0",
  "tree": {
    "$className": "Folder",
    "src": { "$path": "src" },
    "roblox_packages": { "$path": { "optional": "roblox_packages" } },
    "roblox_server_packages": { "$path": { "optional": "roblox_server_packages" } }
  }
}
```

One `$path` per `build_files` entry; `optional` mounts the package's own dependencies only once they exist. The file lands *inside* the installed package — `roblox_packages/.pesde/acme+package/1.0.0/package/` — never in your project root. The last segment is the package's own name: `<scope>+<name>/<version>/<name>`.

## The installed tree

**The same package, under each manager**

```text
Packages/                        <- Wally
  _Index/
    acme_package@1.0.0/
      package/        <- this is src/init.luau
        foo           <- src/foo.luau
        bar           <- src/bar.luau
      dependency

roblox_packages/                 <- pesde
  .pesde/
    acme+package/
      1.0.0/
        package/      <- the package's own name
          src/        <- src/init.luau
            foo       <- src/foo.luau
            bar       <- src/bar.luau
          roblox_packages/
            dependency  <- roblox_packages/dependency.luau
```

`src` is still called `src`, and the package's own dependencies sit in its own `roblox_packages`. The full container is `.pesde/<scope>+<name>/<version>/<name>` — scope and name joined with `+`, then the version, then the name again.

Read those two trees side by side and the design shows: Wally's layout is something the project file *produced*, so you have to know the project file to predict it. pesde's layout is the directory you already have.

> **Keep your code in `src`**
>
> When authoring a package for Roblox, put the source in a `src` directory (or any directory you like) rather than at the root. Because the mapping is 1:1, whatever you choose is what consumers see — a root full of loose files becomes a Roblox folder full of loose files.

**Key points**

- A pesde package has no `default.project.json`; `pesde publish` refuses one and git dependencies ignore it.
- `build_files` is the 1:1 mapping; the consumer's generator writes their tool's config from it, into each installed package.
- The installed tree is your directory tree, under a three-segment container: `roblox_packages/.pesde/<scope>+<name>/<version>/<name>/src`.
- Keep the source in one directory — whatever you choose is what consumers see.
