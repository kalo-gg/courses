# Yanking, deprecating and living with it

*The two levers that exist because deletion does not: yank a version, deprecate a package, and undo either.*

pesde · Publishing to the Registry · 10 min

---

A published version cannot be replaced and a package cannot be removed because you changed your mind. What you have instead are two levers with different scopes.

|  | `pesde yank` | `pesde deprecate` |
| --- | --- | --- |
| Scope | One version — one target of it, or all | The whole package |
| Says | This release is broken | Do not start new work with this |
| Fresh installs | Blocked | Allowed, with a warning |
| Existing lockfiles | Still install fine | Unaffected |
| Reversible | `--undo` | `--undo` |

The distinction that matters is the third row. A yanked version stays installable **from a lockfile**, so yanking a bad release does not break the projects that already depend on it — it only stops the bleeding for everyone else. `pesde deprecate` needs a non-empty reason, which is shown to anyone installing it.

**Both**

```bash
pesde yank "acme/foo@1.0.0 luau"   # that one target
pesde yank acme/foo@1.0.0           # every target of that version
pesde yank --undo "acme/foo@1.0.0 luau"

pesde deprecate acme/foo "superseded by acme/bar"
pesde deprecate acme/foo --undo
```

The argument is `scope/name@version` with an optional target after a space. **Leave the target off and every target of that version is yanked.**

> **Why not just delete it?**
>
> So that the registry stays a reliable source. Any install that worked yesterday works today. The policy is explicit that abandonment is *not* grounds for removal — deprecate it and leave it standing.

## Reading a yank argument

The argument is `scope/name@version` with an optional target after a space, quoted because of that space. Leave the target off and every target of the version is yanked. `--undo` reverses either command, and `-i` picks the index for both. Removal itself is reserved for the registry: forbidden content, security vulnerabilities and legal demands — and a published secret has to be invalidated, because it cannot be unpublished.

**Fixing forward**

```bash
pesde yank "acme/foo@1.1.0 luau"   # stop fresh installs of the broken build
# fix the bug, bump the version, then
pesde publish --dry-run
pesde publish
```

1.1.0 can never be replaced, so the repair is always a new version. Projects that already locked 1.1.0 keep installing it; everyone else resolves 1.1.1.

**Key points**

- `yank` blocks fresh installs of one version — one target, or all if you omit it — and leaves lockfiles working.
- `deprecate` warns on the whole package and needs a reason.
- `--undo` reverses either; `-i` picks the index.
- Removal is for policy, security and legal reasons only; a leaked secret is invalidated, not unpublished.
