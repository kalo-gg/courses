# Realms

*The check a formatter cannot do for you: which containers reach which, and the setting for clones.*

Larvae · Rewriting Requires · 11 min

---

A Roblox place is not one program. Some containers exist only on the server, some are templates cloned to each player, and some are shared. A require that reaches across those boundaries compiles perfectly and fails at runtime, often in front of players. larvae knows the boundaries and checks them.

| Realm | Containers | Meaning |
| --- | --- | --- |
| `ServerOnly` | `ServerScriptService`, `ServerStorage` | Never replicated. A client requiring this cannot work. |
| `StarterClone` | `StarterPlayer`, `StarterGui`, `StarterPack` | Templates. Each player gets a **copy**, so an absolute path here reaches the template, not the running copy. |
| `Shared` | `ReplicatedStorage` and the rest | Reaches both halves. |

The realm of a file is the realm of its mount. larvae reads the Rojo project file to learn that `src/server` lands in ServerScriptService and `src/client` in StarterPlayerScripts, then classifies every require by where both ends land — which is why it can say a client file cannot reach a server one before any of it runs.

> **The Starter subtlety**
>
> This is the one people lose an afternoon to. `@game/StarterGui/Menu` names the *template* sitting in StarterGui. The thing on screen is a clone inside the player's `PlayerGui`, and edits to the template do nothing to it. larvae classifies these containers separately for exactly this reason.

**And the setting that fixes it**

```toml
[requires]
client_relative_requires = true   # off by default
```

This is the payoff. With it on, a require inside the StarterPlayer tree is written as a **relative** path, so the script walks the tree of the copy it is actually running in and reaches its siblings after the clone. It is off by default because it lowers the floor of a relative walk from the mount to the container, and a project that splits StarterPlayerScripts across two mounts asked for that split. A shared file requiring client code is still an error either way.

## Turning the check on and off

**cross_realm, project-wide**

```toml
[requires]
cross_realm = "deny"    # the default: a crossing stops the build
# cross_realm = "allow" # emit every crossing as written
```

`allow` exists for places whose containers Roblox does replicate in ways larvae cannot see. It is an escape hatch, not a default.

**The surgical version**

```luau
-- larvae: allow(cross_realm_require)
local Data = require("@game/ServerStorage/Data")
```

This is the one the project recommends, and the one to reach for first. `cross_realm = "allow"` turns the check off for every file; the comment turns it off for the line that needs it — and it works because the realm check is plumbed through the same flag vocabulary as the lints, which the Flag comments lesson covers.

`larvae check` runs these questions over the whole require graph without writing output, which makes it the natural CI gate. Unresolvable requires and realm violations are always reported there — no level lowers them.

**Key points**

- Three realms: ServerOnly never replicates, StarterClone is a template each player copies, Shared reaches both.
- An absolute path into a Starter container names the template; `client_relative_requires = true` writes the require relative so the clone finds its siblings.
- `cross_realm = "deny"` is the default; `-- larvae: allow(cross_realm_require)` takes the one exception without turning the check off everywhere.
- Realm violations always report in `larvae check`, whatever the levels say.
