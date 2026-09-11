# Three tables

*Three tables, three directories, and a dependency's own dev-dependencies never reach you.*

Wally · The Manifest · 10 min

---

A manifest has three places to put a dependency, and the choice is not a matter of taste: each table writes to a different directory, and Module 4 maps those three directories to three different parts of the datamodel. Putting a package in the wrong one is how a server-only library ends up replicated to every client.

**All three tables**

```toml
[dependencies]
Signal = "sleitnick/signal@^2.0.0"

[server-dependencies]
ProfileStore = "biff/profile-store@^1.0.0"

[dev-dependencies]
TestEZ = "roblox/testez@^0.4.1"
```

Every table is optional and every one has the same syntax. Only the destination differs.

| Table | Installs into | Reaches your consumers? |
| --- | --- | --- |
| `[dependencies]` | `Packages/` | Yes — and they must be able to see it |
| `[server-dependencies]` | `ServerPackages/` | Yes, into their `ServerPackages/` |
| `[dev-dependencies]` | `DevPackages/` | **No.** Never installed for anyone who depends on you |

Three tables, three directories, one rule each. The third row is the one that saves you work.

`wally install` builds all three directories from the manifest and the lockfile, and each one holds the packages named in its table plus whatever those packages pull in. The names are fixed: you cannot rename `ServerPackages/`, and it is Rojo, not Wally, that decides where each directory lands in the datamodel.

**What install writes**

```text
Packages/          <- [dependencies]
ServerPackages/    <- [server-dependencies]
DevPackages/       <- [dev-dependencies]
```

One directory per table, always these three names. A table you did not write produces no directory.

## Dev dependencies stop at you

The rule deserves stating on its own, because it is what makes `[dev-dependencies]` useful rather than merely tidy: **a dependency's own dev-dependencies are never traversed.** Wally walks the graph through the first two tables and stops at the third every time it crosses a package boundary. Your own dev-dependencies are installed for you and for nobody else.

> **Which table for a package you are unsure about?**
>
> Ask what would break if it were missing from a live game. If the game would break, it is a real dependency. If only your tests would, it is a dev dependency. Then ask whether a client is ever allowed to see the code — that is what decides between the first two tables, and the next lesson turns it into a rule.

One consequence worth planning around: because `DevPackages/` is yours alone, it is the right home for anything you would be embarrassed to ship. Test runners, mock builders, one-off tooling — none of it reaches a consumer, and none of it has to be excluded from the published archive later.

**Key points**

- `[dependencies]` installs to `Packages/`, `[server-dependencies]` to `ServerPackages/`, `[dev-dependencies]` to `DevPackages/`.
- The directory names are fixed; Rojo decides where each one lands in the datamodel.
- A dependency's own dev-dependencies are never traversed, so a library's test runner costs you nothing.
- Your dev-dependencies are installed for you and never for a consumer of your package.
- The table you choose is a visibility decision, not a filing decision.
