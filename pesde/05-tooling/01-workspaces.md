# Workspaces

*Several packages in one repository, commands that run on all of them, and four sigils that rewrite themselves.*

pesde · Workspaces, Tooling & CI · 10 min

---

A workspace is one repository holding several pesde packages. Commands run from the root apply to every member, and members may depend on one another.

**The root manifest**

```toml
name = "acme/root"
version = "0.0.0"
private = true

workspace_members = ["pkgs/*"]

[target]
environment = "luau"
```

`private = true` because the root exists to hold the workspace, not to be published.

**A member depending on a sibling**

```toml
# pkgs/foo/pesde.toml
name = "acme/foo"
version = "1.0.0"

[dependencies]
bar = { workspace = "acme/bar", version = "^" }
```

## The version sigils

That bare `^` is the part worth learning. A workspace dependency becomes an ordinary pesde dependency when you publish, and **three of the sigils are filled in with the sibling's real version at that moment**, while two are left exactly as written.

The reason it matters is that a workspace specifier cannot leave the repository. Nobody outside it can resolve `workspace = "acme/bar"`, so publish has to turn it into a registry specifier, and the sigil is how you say what range that should be without hard-coding a version you will forget to bump. Write `^` and every release of a member depends on whatever its siblings were at the moment they were published together.

| You write | Published as (siblings at 2.1.5) | Meaning |
| --- | --- | --- |
| `version = "^"` | `^2.1.5` | Compatible with whatever the sibling was. |
| `version = "~"` | `~2.1.5` | Patch-level updates only. |
| `version = "="` | `=2.1.5` | Exactly this version. |
| `version = "*"` | `*` | Any version. Left alone. |
| `version = "^2.1.0"` | `^2.1.0` | You said it explicitly. Left alone. |

**The other two things the specifier takes**

```toml
[dependencies]
# version omitted entirely — the default is "^"
bar = { workspace = "acme/bar" }

# and a sibling published for several targets needs one naming which
baz = { workspace = "acme/baz", version = "^", target = "luau" }
```

`version` is optional and defaults to `^`, so a bare `{ workspace = "acme/bar" }` behaves exactly like writing the sigil. `target` picks which of the sibling's targets you mean, the same way it does on a registry specifier.

> **Why this is worth having**
>
> It means a monorepo never has to bump cross-references by hand. You raise `acme/bar` to 2.2.0, publish, and every sibling that wrote `"^"` publishes against `^2.2.0` — no sweep, no forgotten file, no release that references a version that does not exist yet.

**Key points**

- A workspace is a `private` root with `workspace_members` globs; root commands run on every member.
- `{ workspace = "acme/bar" }` depends on a sibling; `version` defaults to `^`.
- `^`, `~` and `=` are filled with the sibling's version at publish; `*` and explicit ranges are not.
- `target` on a workspace specifier picks which of the sibling's targets you mean.
