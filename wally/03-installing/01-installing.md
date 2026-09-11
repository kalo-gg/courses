# wally install

*One command reads the manifest, wipes three directories and writes them again from scratch.*

Wally · Installing and the Tree · 10 min

---

Everything in Module 2 was a file you write. `wally install` is the command that reads it. It does four things in order — resolve a version for every package in the graph, write `wally.lock`, delete the package directories, and download and lay out what it resolved — and it narrates each of them on the way past.

**A clean install**

```bash
$ wally install
   Resolved 3 dependencies
  Generated lockfile
    Cleaned package destination
 Downloaded biff/minimal@0.1.0
[INFO]  Downloaded 3 packages!
```

One line per phase, then one line per package it had to fetch. The ragged indentation is Wally's, not a paste error.

The first line is the one worth reading carefully. **The resolved count excludes the root package** — your own project is in the graph, but it was never something that had to be found — so `Resolved 3 dependencies` on a manifest listing two entries means one of those two brought a package of its own along.

| Phase | What happens | The line |
| --- | --- | --- |
| Resolve | Walk the graph and choose a version for every package | `   Resolved 3 dependencies` |
| Lock | Write the resolved graph to `wally.lock` | `  Generated lockfile` |
| Clean | Delete `Packages/`, `ServerPackages/` and `DevPackages/` | `    Cleaned package destination` |
| Download | Fetch each package and write the link modules | ` Downloaded biff/minimal@0.1.0` |

The third phase is the one nobody expects, and it is the subject of the next callout.

> **Install deletes the package directories first**
>
> `Cleaned package destination` is literal. Install removes `Packages/`, `ServerPackages/` and `DevPackages/` **entirely** before it writes anything, so a module you dropped in there by hand is gone without a warning. It is also why `wally-package-types` has to be re-run after every install: whatever it rewrote last time no longer exists. Treat all three directories as build output and keep nothing of your own in them.

## The only flag

**Installing a project somewhere else**

```bash
$ wally install --project-path ./game
   Resolved 3 dependencies
  Generated lockfile
    Cleaned package destination
[INFO]  Downloaded 3 packages!
```

`--project-path` points at the directory holding `wally.toml`. It is `install`'s only flag in 0.3.2.

There is no `--locked`. The README on Wally's default branch documents one, the release does not have it, and the README at the v0.3.2 tag says as much itself. If you want CI to fail rather than quietly re-resolve, check that the lockfile did not move — `git diff --exit-code wally.lock` after the install — because the flag that would have done it for you is not there.

Install is safe to repeat: the same four phases run every time, and with an unchanged manifest and lockfile they produce an identical tree. That is the point of it, and it is what makes the destructive clean defensible — the directories can always be rebuilt from two files you did commit.

**Key points**

- `wally install` resolves, writes `wally.lock`, cleans the package directories and downloads — in that order, one line each.
- The resolved count excludes the root package, so it exceeds your manifest whenever a dependency has dependencies of its own.
- Install **deletes** `Packages/`, `ServerPackages/` and `DevPackages/` before writing them. Keep nothing of your own there.
- Anything that rewrites the installed tree, `wally-package-types` above all, has to run again after every install.
- `--project-path` is the only flag in 0.3.2; `--locked` is documented on the default branch and is not in the release.
