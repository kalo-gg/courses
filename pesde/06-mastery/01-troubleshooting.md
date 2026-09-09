# Troubleshooting resolution and linking

*The messages pesde prints when something is wrong, what each one really means, and the fix for each.*

pesde · Best Practices & Capstone · 10 min

---

Most pesde failures are one of seven, and four of them do not look like what they are: a Wally-index error that is really a hyphen, a types warning that is really a missing entry point, a package that syncs as an empty folder because a script path was wrong, and a publish refusal about the lockfile that only wants you to run install.

| You see | It means | Do |
| --- | --- | --- |
| `no wally index found` after `pesde add acme/my-package` | The hyphen made pesde parse a Wally name | Rename with an underscore |
| `no sourcemap_generator found in project. wally types will not be generated`, and the package requires as `nil` | No entry point for the Wally dependency | Wire `[scripts] sourcemap_generator` |
| Packages sync into Studio as empty folders | `[scripts]` path wrong, or `install` never ran | Fix the path, `pesde install`, then serve |
| `outdated lockfile, please run the install command first` | Manifest edited after the last install | `pesde install` |
| `pesde x` cannot resolve a package that exists | It is published only for Lune | `pesde x -t lune …` |
| `warn: package … requires lune ^0.8.9, but 0.9.0 is installed` | Engine mismatch | Adjust `[engines]` or the runtime |
| `roblox packages may not depend on non-roblox packages` | A Lune tool in `[dependencies]` | Move it to `[dev_dependencies]` |

**The messages, verbatim**

```text
no wally index found
no sourcemap_generator found in project. wally types will not be generated
outdated lockfile, please run the install command first
roblox packages may not depend on non-roblox packages
no build files found in target
roblox doesn't support bin exports!
```

Read each one for its noun. `wally` means a hyphen in a name you meant for pesde; `types` means no entry point at all; `lockfile` means run install.

## When the tree is wrong

Every installed file comes from the store, so deleting a packages directory and reinstalling is always safe. The ladder is: `pesde install`; then `pesde install --force`, which reinstalls every package even if it looks present; then `pesde cas prune` to drop what nothing references. `pesde cas path` prints where the store is, and `PESDE_HOME` moves all of it.

Alias rules are the other quiet source of surprises. An alias is `[a-zA-Z0-9_-]+` and at most 48 characters, and a handful of names are refused outright: `self`, `cart`, the engine names (`pesde` and `lune`) and Windows' reserved file names. A rejected alias is an error at `add` time, not a mystery later — but a *legal* alias that collides with a `[scripts]` entry is exactly the mystery `pesde run` resolves in the package's favor.

**The repair ladder**

```bash
pesde install            # 1. the usual
pesde install --force    # 2. reinstall every package
pesde cas prune          # 3. drop what nothing references
pesde cas path           # where the store lives, if you must look
```

**Key points**

- Read the message for its noun: `wally` means a hyphen, `types` means no entry point, `lockfile` means run install.
- Empty package folders in Studio mean the generator never ran — check the `[scripts]` path, then install before serving.
- `install --force`, then `cas prune`; everything reinstalls from the store.
- Aliases: `[a-zA-Z0-9_-]+`, ≤ 48 characters; `self`, `cart`, the engine names and Windows' reserved file names are refused.
