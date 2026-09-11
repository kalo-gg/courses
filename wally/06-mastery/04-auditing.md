# Auditing a dependency

*Read the index entry, the archive and the realm before a package becomes yours to maintain.*

Wally · Best Practices & Capstone · 9 min

---

Adding a dependency is a maintenance commitment, and Wally makes it a one-line commitment, which is the problem. Three things are worth reading before that line goes in: the package's **index entry**, which tells you what you are really getting; its **archive**, which tells you what is actually in it; and its **realm**, which decides where it is even allowed to live in your project. All three are public, and none takes more than a minute.

**The index entry for biff/drop-table**

```json
{"package":{"name":"biff/drop-table","version":"2.3.1","registry":"https://github.com/UpliftGames/wally-index","realm":"server","description":"Weighted drop tables","license":"MIT"},"dependencies":{"Signal":"sleitnick/signal@>=2.0.0, <3.0.0"},"server-dependencies":{},"dev-dependencies":{}}
{"package":{"name":"biff/drop-table","version":"2.9.0","registry":"https://github.com/UpliftGames/wally-index","realm":"server","description":"Weighted drop tables","license":"MIT"},"dependencies":{"Signal":"sleitnick/signal@>=2.0.0, <3.0.0"},"server-dependencies":{},"dev-dependencies":{}}
{"package":{"name":"biff/drop-table","version":"3.0.0","registry":"https://github.com/UpliftGames/wally-index","realm":"server","description":"Weighted drop tables","license":"MIT"},"dependencies":{"Signal":"sleitnick/signal@>=3.0.0, <4.0.0"},"server-dependencies":{},"dev-dependencies":{}}
```

Three published versions, oldest first. Everything you need is on these lines: the realm, the dependencies, and the fact that 3.0.0 moved to a new major of Signal.

Start with the version you would actually get. A requirement is not a version, and **a bare version is a caret** — `"biff/drop-table@2.3.1"` means `^2.3.1`, which is `>=2.3.1, <3.0.0`. The index will not show you a caret at all, because Wally normalises requirements on the way in: whoever wrote `^2.0.0` for Signal above sees `>=2.0.0, <3.0.0` stored. Reading the entry with that in mind is the difference between knowing which version you have and assuming it.

## What is actually in the archive

**wally package --list**

```bash
$ wally package --list
wally.toml
README.md
default.project.json
src/init.luau
src/Table.luau
src/Weights.luau
tests/init.spec.luau
tests/fixtures/big-drop-table.rbxm
```

Eight entries, and two of them are tests. The `.rbxm` fixture is the kind of file that walks a package towards the 2 MiB cap without anyone noticing.

That listing is the only honest view of a package. `include` looks like a whitelist and is not: a file is kept if `include` matches it, **and otherwise kept unless `exclude` matches it**, so a short `include` list shrinks nothing. A published package with six `include` entries can ship 270 files. When you audit someone else's package, `wally package --list` on a clone of it tells you what you would be downloading; when you publish your own, it is what you run before the first `wally publish`.

> **You inherit its dependencies, but not its tests**
>
> A package's own `[dependencies]` become part of your tree — the entry above brings `sleitnick/signal` with it whether you wanted it or not, and the resolver will pick **one** semver-compatible version of Signal for your whole graph. Its `[dev-dependencies]` do not: a dependency's dev table is never traversed, so a test runner buried in a library you depend on costs you nothing. That is where the audit stops.

## Which realm it declares

The last line to read is `realm`. It describes the package, and it constrains *you*: `[dependencies]` accepts only packages whose manifest says `realm = "shared"`, while `[server-dependencies]` and `[dev-dependencies]` accept any realm. `biff/drop-table` declares `server`, so it cannot go in `[dependencies]` — and if you put it there, the error will not say "realm". The candidate is filtered out of the pool before matching, so you get `No packages were found that matched (Shared) biff/drop-table@…` followed by `Are you sure this is a Shared dependency?`. That second line is the real symptom.

**Key points**

- Read the index entry before you depend on a package: it is one line of JSON in a git repository anyone can clone.
- The requirement you write is not the version you get — a bare version is a caret, and the index only ever shows the expanded form.
- `wally package --list` is the only honest view of an archive, because `include` is not a whitelist.
- You inherit a package's `[dependencies]` and not its `[dev-dependencies]`, and one version of each shared package is picked for the whole graph.
- The realm a package declares decides which of your three tables may hold it, and a mistake there reads as a missing package.
