# Five ways to name a dependency

*Registry, Wally, git, workspace and path specifiers, and when each one is the right reach.*

pesde · The Manifest · 11 min

---

Every entry under `[dependencies]` is an **alias** on the left and a **specifier** on the right. The alias is what you require and what `pesde remove` takes; the specifier says where the package comes from. Which of the distinguishing keys is *present* — `name`, `wally`, `repo`, `workspace` or `path` — decides which kind it is. Order inside the inline table is irrelevant, so `{ version = "1.2.3", name = "acme/foo" }` is a perfectly good pesde specifier.

**All five**

```toml
[dependencies]
# a pesde registry package
foo = { name = "acme/foo", version = "1.2.3", index = "acme", target = "lune" }

# a Wally package — Roblox projects only
bar = { wally = "acme/bar", version = "2.3.4", index = "acme" }

# a git repository, optionally a subdirectory of one
baz = { repo = "acme/packages", rev = "aeff6", path = "pkgs/baz" }

# another package in the same workspace
qux = { workspace = "acme/qux", version = "^" }

# a directory on this machine
dev = { path = "/home/user/foo" }
```

| Kind | Key | Reach for it when |
| --- | --- | --- |
| pesde | `name` | The package is published. The normal case. |
| Wally | `wally` | You need a Roblox library that only exists on Wally. |
| git | `repo` | There is no release yet, or you need an unpublished fix. |
| workspace | `workspace` | It is a sibling package in this repository. |
| path | `path` | You are developing both sides at once, locally. |

## git and path

A **git** dependency's `repo` is either `owner/name` for GitHub or a full URL, and `rev` is a tag, a branch or a commit hash — pesde's own examples use `rev = "main"`. A branch is convenient and moving, a commit is exact; pick according to whether you want the dependency to drift.

If the repository holds several packages, `path` picks one — and what it points at must be a valid pesde *or* Wally package. A git dependency has no version to resolve, so what a lockfile pins is the commit it read.

> **Path dependencies cannot be published**
>
> They point at a directory on one machine, so a published package containing one would be uninstallable for everyone else. pesde rejects it at publish time rather than letting you find out from a bug report. The path must be absolute and hold a `pesde.toml`.

> **Wally dependencies need a sourcemap generator to work at all**
>
> A Wally package has no `pesde.toml`, so nothing in it declares a `lib`. pesde finds the entry point by running your `sourcemap_generator` script under `[scripts]` and taking the first Luau file out of the sourcemap it prints. Without that script the warning is only *"no sourcemap_generator found in project. wally types will not be generated"*, but the linker is left pointing at a placeholder path that does not exist — so you lose the entry point, not merely the types. Roblox Projects wires this up.

**`pesde add` spellings**

```bash
pesde add pesde/hello                              # registry
pesde add pesde/hello@1.2.3                        # a version
pesde add wally#acme/bar                           # Wally
pesde add gh#acme/package#main                     # GitHub
pesde add https://git.acme.local/pkg.git#aeff6     # any git URL
pesde add workspace:acme/qux                       # workspace sibling
pesde add path:/home/user/foo                      # local directory
```

Plus `--peer`, `--dev`, `--alias`, `--index` and `--target`.

**Key points**

- Alias on the left, specifier on the right; the present key (`name` / `wally` / `repo` / `workspace` / `path`) decides the kind.
- git for unreleased code, path for side-by-side development — and path never publishes.
- A Wally dependency has no manifest, so it needs your `sourcemap_generator` to get an entry point at all.
- Every kind has a `pesde add` spelling: `wally#`, `gh#`, a URL, `workspace:`, `path:`.
