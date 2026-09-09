# Dev, peer, and where to look

*Three dependency tables, what each means for a consumer, and naming more than one registry.*

pesde · The Manifest · 10 min

---

Not every dependency ships to consumers. The table you put a dependency in is a statement about who installs it: everyone, only you, or the consumer on your behalf.

| Section | Installed for consumers? | For |
| --- | --- | --- |
| `[dependencies]` | Yes | What the package needs to run. |
| `[dev_dependencies]` | No | Test runners, build tools, linters. |
| `[peer_dependencies]` | No — the consumer must supply it | A shared thing there must be exactly one of. |

A peer dependency is the subtle one. It says *this package needs `acme/foo`, but it must be the same `acme/foo` the rest of the project is using* — the classic case being a framework whose plugins must share its instance rather than each installing a private copy.

**Adding to the right table**

```bash
pesde add --dev acme/test_runner
pesde add --peer acme/framework
```

> **`--prod` and `--dev` at install time**
>
> `pesde install --prod` links everything except dev dependencies, which is what a release build wants. `--dev` does the opposite and links only them.

## Indices

**More than one registry**

```toml
[indices]
default = "https://github.com/pesde-pkg/index"
acme = "https://github.com/acme/pesde-index"

[wally_indices]
default = "https://github.com/UpliftGames/wally-index"

[dependencies]
internal = { name = "acme/internal", version = "1.0.0", index = "acme" }
```

A dependency with no `index` uses `default`. Wally dependencies read `[wally_indices]` instead.

The two tables are separate because they are separate registries with separate formats. A dependency's `index` key is resolved against whichever table matches its kind, so the same alias `acme` can name a pesde index and a Wally index without ambiguity.

**Key points**

- `[dependencies]` for consumers, `[dev_dependencies]` for you, `[peer_dependencies]` for the thing there must be one of.
- `pesde install --prod` skips dev dependencies; `--dev` links only them.
- `[indices]` and `[wally_indices]` are separate tables for separate registries; a specifier's `index` names an entry in the matching one.
- No `index` means `default`.
