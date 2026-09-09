# Day to day

*Listing, updating, removing by alias, and the content-addressable store under every project.*

pesde · pesde Foundations · 11 min

---

Once a project has dependencies, five commands cover an ordinary week. Only one of them takes a package *name*; the rest take the alias you gave it.

| Command | What it does |
| --- | --- |
| `pesde list` | The project's dependencies. |
| `pesde outdated` | Which of them have newer versions. |
| `pesde update` | Re-resolve and install. `--no-install` touches only the lockfile. |
| `pesde remove <alias>` | Drop a dependency, by the alias it was added under. |
| `pesde install --force` | Reinstall everything, for when a tree has gone wrong. |

**An ordinary week**

```bash
pesde list                # the aliases and specifiers in the manifest
pesde outdated            # which of them have newer versions
pesde update              # re-resolve inside the manifest's ranges
pesde remove log          # by alias, not by package name
pesde install --force     # reinstall everything, present or not
```

`pesde add` is the only one of the six that takes a package name; every command here takes the alias. `pesde outdated` compares against the newest published version and ignores the range you declared — pass `--strict` to be told only about versions your range would actually accept.

Note that `remove` takes the **alias**, not the package name. The alias is the key on the left of the manifest entry, which defaults to the package name but can be anything — `pesde add pesde/hello --alias greet` gives you `greet`, and that is what you require and what you remove. An alias is made of letters, digits, underscores and hyphens (`[a-zA-Z0-9_-]+`) and is at most 48 characters. Some names are refused: `self` and `cart`, the engine names (`pesde`, `lune`), and Windows' reserved file names.

## The content-addressable store

Downloaded package contents live once in a **CAS** — a content-addressable store under pesde's home directory — and each project links to it rather than holding its own copy. Ten projects depending on the same version cost one download and one copy on disk.

**Housekeeping**

```bash
pesde cas path      # where this location's store lives
pesde cas prune     # drop files and packages nothing references
pesde self-upgrade  # update pesde itself
```

`PESDE_HOME` overrides where all of this lives; it defaults to `$HOME/.pesde`.

**Key points**

- `list`, `outdated`, `update`, `remove <alias>`, `install --force` — and only `add` takes a package name.
- An alias is `[a-zA-Z0-9_-]+`, ≤ 48 characters; `self`, `cart`, the engine names and Windows' reserved file names are refused.
- `pesde outdated` ignores your declared range unless you pass `--strict`.
- Package contents live once in the CAS under `PESDE_HOME`; `pesde cas prune` reclaims space, and `pesde install --locked` is the CI form.
