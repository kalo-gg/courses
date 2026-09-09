# Snapshots

*Record what a value serialized to, review the file, and know when a stored key is stale.*

Lest · The Loop and CI · 11 min

---

A snapshot test records what a value serialized to the first time it ran, then fails if that ever changes. It is the right tool when asserting on a large structure field by field would be tedious and unreadable — rendered output, config resolution, a parse tree.

| State | What happens |
| --- | --- |
| No stored value | Written, and the test **passes** |
| Stored value matches | **Passes** |
| Stored value differs | **Fails**, with a diff |
| `lest -u` and it differs | Overwritten, and the test **passes** |

> **The first run is meant to pass — so you have to read the file**
>
> A snapshot committed without being reviewed records whatever the code did that day, **bug included**, and every future run then defends that bug. Snapshots live beside the spec in `__snapshots__/`, they are plain readable text, and they are part of the test: commit them, and review them like any other assertion.

## Keys and hints

**Several in one test**

```luau
it('renders both modes', function ()
	expect(render(doc)).toMatchSnapshot('default')
	expect(render(doc, { compact = true })).toMatchSnapshot('compact')
end)
```

A hint replaces the automatic number — more readable in the file, and stable if you reorder the calls.

Each snapshot is stored under a key built from the test's identity — the describe path and the test name — plus a per-test counter starting at 1. A hint replaces the counter, never the path, so `default` and `compact` above read as names rather than as `1` and `2`, and reordering the two calls does not swap their stored values. The framework only serializes: losslessly, with no depth cap, refusing cycles and non-scalar keys. The CLI owns the file, the comparison and `-u`.

**What the sandbox prints instead of a file**

```text
Snapshots:   2 total
  render > renders both modes default: { body = { "widget x2", "gadget x1" }, title = "Receipt" } (written; the CLI owns comparison)
  render > renders both modes compact: { lines = 2, title = "Receipt" } (written; the CLI owns comparison)
```

The CLI would write these under `__snapshots__/` beside the spec and diff the next run against them; the browser has no disk, so it shows you what it would have stored.

## Obsolete keys

When a stored key no longer corresponds to any test that ran, Lest reports it as **obsolete** rather than silently keeping it, and `lest -u` prunes them. Detection is skipped on a filtered run — `-t`, `--changed`, or watch mode — because there "stored but not produced" only means that test did not run this time, and pruning would delete live snapshots. Running one suite by name is *not* filtered: every test in it executes.

> **One more narrowing, and it is the one you will hit**
>
> Detection compares against the `.snap` files of spec files that produced **at least one snapshot this run**. A spec whose tests have all stopped calling `toMatchSnapshot()` never opens its `.snap` — so every key stranded in there goes unflagged, forever, on an unfiltered run. Delete the last snapshot assertion from a spec and you have to delete its `.snap` yourself.

**Key points**

- First run writes and passes; matching passes; different fails with a diff; `-u` overwrites.
- Snapshots live in `__snapshots__/`, are plain text, and are committed and reviewed like any assertion.
- A key is the test's identity plus a counter; a hint replaces the counter.
- Obsolete keys are reported on unfiltered runs, and only for specs that still snapshot something.
