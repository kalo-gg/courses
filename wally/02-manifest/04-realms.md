# Realms

*shared, server and dev, and the rule that catches people is not the one the docs state.*

Wally · The Manifest · 9 min

---

`realm` is the fourth required key in `[package]`, and it takes one of three words: `shared`, `server` or `dev`. It is the smallest field in the manifest and the one that produces the most confusing error in the tool, so it is worth being precise about what it actually does.

**Declaring your realm**

```toml
[package]
name = "biff/profile-store"
version = "1.0.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "server"
```

This package has declared itself server code. Notice what that does *not* say: nothing here is about where its own dependencies go.

| Realm | The package is | A consumer may hold it in |
| --- | --- | --- |
| `shared` | Code either side may run | Any of the three tables |
| `server` | Code only the server may see | `[server-dependencies]` or `[dev-dependencies]` |
| `dev` | Tooling, not shipped code | `[server-dependencies]` or `[dev-dependencies]` |

Read the third column as the rule. Your realm is a statement about *other people's* manifests.

Here is the rule that matters, and it is not the one people expect: **the dependency table constrains, not your own `realm`.** Anything you list in `[dependencies]` must be a package whose own manifest says `realm = "shared"`. `[server-dependencies]` and `[dev-dependencies]` accept a package of any realm at all.

So your own `realm` never limits what you can depend on. Declaring `realm = "server"` does not let you put server packages in `[dependencies]`, and declaring `realm = "shared"` does not stop you depending on a server package — you just have to put it in a table that accepts one. Your realm is purely a message to consumers about which of *their* tables may hold *you*.

**The same line, twice**

```toml
# biff/profile-store declares realm = "server"

[dependencies]
ProfileStore = "biff/profile-store@^1.0.0"    # filtered out: not shared

[server-dependencies]
ProfileStore = "biff/profile-store@^1.0.0"    # accepted: any realm
```

Identical spec, identical package, opposite outcomes. The only thing that changed is the table header above it.

## When it goes wrong

> **A realm mistake does not say "realm"**
>
> Put a server package in `[dependencies]` and the candidate is filtered out of the pool **before** matching, so the resolver reports the only thing it can see: that nothing matched. You get `No packages were found that matched (Shared) biff/server@>=1.0.0, <2.0.0.` followed by `Are you sure this is a Shared dependency?` The first line reads like a typo in the package name. The second line is the actual diagnosis, and it is the one to read.

So when an install says a package was not found and you are certain the name is right, check the realm before you check anything else. The fix always has the same shape: move the line into `[server-dependencies]`, leaving the alias and the requirement exactly as they were.

**Key points**

- `realm` is `shared`, `server` or `dev`, and the three install directories mirror the three tables.
- The rule is about the **table**: `[dependencies]` accepts only shared packages, the other two accept any realm.
- Your own `realm` constrains nobody but your consumers — it never limits what you may depend on.
- A realm mistake reports a missing package; `Are you sure this is a Shared dependency?` is the real symptom.
- The fix is to move the line to a table that accepts the realm, leaving alias and requirement alone.
