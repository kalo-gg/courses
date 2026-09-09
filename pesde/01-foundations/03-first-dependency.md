# Your first dependency

*Add a package, install it, require it, and commit the lockfile that makes the install repeatable.*

pesde · pesde Foundations · 10 min

---

A dependency is two commands. `pesde add` edits the manifest and nothing else; `pesde install` resolves versions, downloads what it needs into pesde's store and links the result into a packages directory next to your code.

Keeping them separate is deliberate: `add` is a decision, `install` is the work, and the lockfile records what the work produced. It is also why a manifest edited by hand is exactly as good as one edited by `add` — nothing about the tree has happened yet either way.

**Two commands**

```bash
pesde add pesde/hello
pesde install
```

`add` edits the manifest. `install` resolves, downloads and links.

**What `add` wrote**

```toml
[dependencies]
hello = { name = "pesde/hello", version = "^1.0.0" }
```

The **alias** on the left is yours to choose (`--alias greet`); the **specifier** on the right names the package. `add` picks `^` of the newest version.

## Install, link, require

`add` writes `hello = { name = "pesde/hello", version = "^1.0.0" }` into `[dependencies]`. `install` then creates two things: a `luau_packages/` directory holding the linked package, and a **`pesde.lock`** recording the exact versions resolved. The file in `luau_packages/` is a **linker**: a small module pesde writes under the alias that requires the real package, which lives once in the store, and re-exports its types (since 0.7.3 that includes type functions). It is two lines — `local module = require(<path>)` and `return module` — with no comment header of its own. You require the linker by path, never the store.

**Requiring it**

```luau
local hello = require("./luau_packages/hello")

hello()
-- pesde run
-- Hello, pesde! (pesde/hello@1.0.0, luau)
```

> **Commit the lockfile**
>
> `pesde.lock` is what makes an install reproducible. Commit it, and use **`pesde install --locked`** in CI — it fails rather than silently resolving something new when the lockfile has drifted from the manifest.

**Key points**

- `pesde add` writes the manifest; `pesde install` resolves, downloads, links and writes `pesde.lock`.
- The alias is what you require and what `remove` takes; the specifier is where the package comes from.
- `luau_packages/<alias>.luau` is a two-line linker into the store — `local module = require(…)`, `return module` — and re-exports the package's types.
- Commit `pesde.lock`; `pesde install --locked` in CI fails instead of resolving something new.
