# Publishing it

*Dry run, publish, the flags, multi-target releases, and the docs the registry page shows.*

pesde · Publishing to the Registry · 10 min

---

Two commands, in that order — and the second one cannot be taken back.

**Publishing**

```bash
pesde publish --dry-run   # build the tarball, upload nothing
pesde publish             # for real, after a confirmation prompt
```

| Flag | Effect |
| --- | --- |
| `-d, --dry-run` | Produce the tarball that *would* be published. |
| `-y, --yes` | Skip the confirmation prompt. |
| `-i, --index` | Publish to a named index rather than `default`. |
| `--no-verify` | Skip syntax validation of the package's exports. |

> **This is permanent**
>
> Once published, a version can never be replaced and the package cannot be removed because you have changed your mind. Removal happens only for policy reasons — prohibited content, security vulnerabilities, or a legal requirement. A published secret has to be **invalidated**, because it cannot be unpublished.

## Multi-target releases

The same name and version may be published several times under different targets — that is how one library serves Roblox and Lune. One constraint: **versions sharing a version number must share a description.** In practice that is one source tree published twice with a different `[target]` each time — the Roblox one carrying `build_files`, the Luau one not. pesde documents the rule but not the procedure; there is no `--target` flag on `publish`, so the manifest's `[target]` is edited between the two runs and the second publish is an ordinary `pesde publish` of the same version.

## Documentation

The `README.md` at the root is shown on the package's registry page. A `docs/` directory of markdown files becomes a documentation site alongside it — with `sidebar_position` frontmatter to order pages, and `_category_.json` inside a subdirectory to label and place a section.

**Ordering the sidebar**

```text
---
sidebar_position: 2
---

# Getting Started
```

And `docs/guides/_category_.json`: `{ "label": "Guides", "position": 3 }`.

> **Docs need to be in `includes` too**
>
> It is the most common version of the whitelist mistake, because everything looks right locally. Add `"docs/**/*.md"` or the registry page will have no documentation on it. This one is only a warning — `docs directory not included, consider adding one`, alongside the same warning for a missing README — so a publish with no docs succeeds, and the gap shows up on the registry page instead.

**Key points**

- `--dry-run`, then `publish`; `-y` skips the prompt, `-i` picks the index, `--no-verify` skips export validation.
- A version is forever: fix forward, and yank the bad one.
- Same name and version may be published once per target, with the same description.
- README is the registry page; `docs/**/*.md` becomes a docs site — if `includes` lists it.
