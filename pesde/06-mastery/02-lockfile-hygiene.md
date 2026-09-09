# The lockfile and reproducible installs

*What pesde.lock records, which commands touch it, and the habits that keep every machine installing one tree.*

pesde · Best Practices & Capstone · 9 min

---

`pesde.lock` records what the last `install` resolved: each package's exact version *and* target, and the dependency graph between them, so the next install reproduces the tree without resolving again. It carries a generated-file marker so diff tools collapse it. You never edit it by hand — the manifest is what you edit, and the lockfile is what the tools derive from it.

**The request the lockfile answers**

```toml
[dependencies]
promise = { name = "acme/promise", version = "^4.0.0" }
signal = { wally = "sleitnick/signal", version = "^2.0.0" }
```

Two ranges and two registries. `pesde.lock` is the answer `install` wrote down: the exact version *and* target it chose for each, plus everything those two pulled in behind them.

| Command | pesde.lock |
| --- | --- |
| `pesde add` | Untouched — only the manifest changes |
| `pesde install` | Written or updated to match the manifest |
| `pesde install --locked` | Refuses to re-resolve, and fails if it is behind the manifest — it still writes the file back, byte for byte identical |
| `pesde update` | Re-resolved from scratch — every dependency, not only the ones whose requirement changed — then installed |
| `pesde update --no-install` | Re-resolved and rewritten, nothing installed |
| `pesde outdated` | Read, compared with the registries — against the newest published version, not your declared range, unless you pass `--strict` |
| `pesde publish` | Refused if it is behind the manifest — a check that runs for `roblox` and `roblox_server` targets only |

## Habits

Commit the lockfile. Update deliberately: `pesde outdated` to see what moved, `pesde update --no-install` to rewrite only the lockfile, read the diff, then `pesde install --locked` to prove CI will accept it, and commit the manifest and lockfile together.

On a merge conflict in `pesde.lock`, do not resolve the conflict by hand and do not just run `install`: a lockfile with conflict markers in it is not parseable, and `pesde install` aborts on it — a *missing* lockfile is the only case install recovers from on its own. Delete `pesde.lock`, then run `pesde install`, and the file comes back resolved from the merged manifest. Remember too that yanked versions still install from a lockfile — the yank blocks fresh resolution only, so a `--locked` build keeps working after one, which is a reason to run `outdated` on a schedule rather than waiting for a failure.

**A deliberate update**

```bash
pesde outdated             # what moved on
pesde update --no-install  # rewrite the lockfile only
git diff pesde.lock        # read it
pesde install --locked     # what CI will do
```

**Key points**

- The lockfile holds exact versions and targets plus the graph; `install` writes it, `--locked` rewrites it unchanged rather than re-resolving, and `publish` refuses a stale one — for `roblox` and `roblox_server` targets.
- `add` never touches it; `update --no-install` touches only it.
- Update on purpose: outdated, update, diff, locked install, one commit.
- A conflicted lockfile is deleted, not merged: `install` aborts on one, and only a missing lockfile is recovered.
- Yanks do not break locked builds — so check `outdated` on a schedule.
