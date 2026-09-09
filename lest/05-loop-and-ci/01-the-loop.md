# The save-run loop

*Watch mode follows the require graph rather than a glob, and --changed does the same from a git ref.*

Lest · The Loop and CI · 10 min

---

Watch mode is **graph-driven, not glob-driven**. Saving a file invalidates its content hash; the inverted require graph then yields exactly the specs whose transitive requires touched it, and only those re-run. Editing one helper does not re-run a suite that never required it.

**The tight inner loop**

```bash
lest --watch
lest run unit --watch
lest run unit -t 'parser' --watch
```

- `cloud`, `studio` and `gargantuan` suites are always excluded — the fast loop never waits on a network round-trip or an engine boot.
- Hidden files are ignored, except `.luaurc` and `.config.luau`, which affect resolution.
- Saving `lest.toml` reloads the configuration; a config that fails to load is rejected with a warning and the previous one stays live.
- If the file watcher itself stops, that is a tool error, not a silent exit that looks like a clean quit.

## `--changed`

**The same idea, from git**

```bash
lest --changed origin/main
lest --changed HEAD~1
```

Diff against a ref, then walk the same inverted require graph.

The same inverted graph, seeded from a git diff instead of a save: every file the ref differs on is treated as changed, and every spec that transitively requires one of them runs. It is the right shape for a large repository's pull-request job, where re-running everything for a one-line change is minutes nobody needed.

> **Nothing affected is an answer, not a mistake**
>
> When a change touches no spec's require graph, the run exits **0** with a note, and any coverage minimum is explicitly skipped. `--changed` also conflicts with `--watch` — watch mode already selects by change, from the file system rather than from git, so combining them is rejected rather than accepted and ignored.

> **A filter that matches nothing is a tool error**
>
> `lest -t 'crat'` exits **2**, not 0. A typo'd `-t` in CI must not look like a passing suite — which is the same instinct behind everything in this module.

**Key points**

- `--watch` re-runs the specs whose require graph touched the saved file, and only those.
- Engine suites are always excluded from the loop; `lest.toml` hot-reloads and a bad reload is rejected.
- `--changed <ref>` walks the same graph from a git diff; nothing affected exits 0 with a note.
- `--changed` and `--watch` do not combine; `-t` matching nothing is exit 2.
